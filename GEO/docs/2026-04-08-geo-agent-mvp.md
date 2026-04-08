# GEO Agent MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an MVP GEO article agent that accepts a keyword plus product context and knowledge sources, then generates an evidence-backed article draft with review output.

**Architecture:** Use a small FastAPI backend with explicit pipeline stages: request intake, keyword understanding, article planning, evidence retrieval from uploaded knowledge files, draft generation, and review. Persist job metadata and generated artifacts in SQLite, while storing uploaded knowledge files on disk for simple local retrieval. Keep the first version API-only so we can validate generation quality before adding a UI.

**Tech Stack:** Python 3.11, FastAPI, Pydantic, SQLAlchemy, SQLite, pytest, httpx, local Markdown/JSON knowledge files, one LLM provider adapter

---

## File Structure

- `pyproject.toml`
  Python project metadata and dependencies.
- `src/geo_agent/main.py`
  FastAPI app bootstrap and route registration.
- `src/geo_agent/config.py`
  Environment-backed settings for storage paths, database URL, and LLM provider config.
- `src/geo_agent/db.py`
  SQLAlchemy engine, session factory, and base model setup.
- `src/geo_agent/models.py`
  Persistent job and artifact tables.
- `src/geo_agent/schemas.py`
  API request and response models.
- `src/geo_agent/routes/jobs.py`
  Endpoints for creating a generation job and fetching results.
- `src/geo_agent/services/keyword_understanding.py`
  Search intent, funnel stage, and risk classification.
- `src/geo_agent/services/content_planning.py`
  Content angle selection and article outline generation.
- `src/geo_agent/services/knowledge_base.py`
  Local knowledge source registration and chunk extraction.
- `src/geo_agent/services/evidence_retrieval.py`
  Retrieve relevant evidence chunks for a requested angle.
- `src/geo_agent/services/article_generation.py`
  Produce article draft from plan plus evidence.
- `src/geo_agent/services/review.py`
  Review draft for repetition, unsupported claims, and GEO-structure rules.
- `src/geo_agent/services/pipeline.py`
  Orchestrate the end-to-end MVP flow.
- `src/geo_agent/llm/base.py`
  LLM client protocol.
- `src/geo_agent/llm/mock.py`
  Deterministic fake LLM for tests.
- `tests/test_jobs_api.py`
  End-to-end API behavior tests.
- `tests/test_keyword_understanding.py`
  Unit tests for keyword analysis logic.
- `tests/test_content_planning.py`
  Unit tests for article planning logic.
- `tests/test_knowledge_base.py`
  Unit tests for source registration and chunking.
- `tests/test_review.py`
  Unit tests for review rules.
- `storage/knowledge/`
  Uploaded or copied source files for the MVP.

## Scope Check

This plan covers one subsystem: an API-first content generation MVP. It intentionally excludes web UI, analytics dashboards, multi-tenant auth, browser automation, scheduled publishing, and production deployment. Those should be separate follow-up plans once the generation loop is proven.

### Task 1: Bootstrap the API Skeleton

**Files:**
- Create: `pyproject.toml`
- Create: `src/geo_agent/__init__.py`
- Create: `src/geo_agent/main.py`
- Create: `src/geo_agent/config.py`
- Create: `src/geo_agent/db.py`
- Create: `src/geo_agent/schemas.py`
- Create: `src/geo_agent/routes/__init__.py`
- Create: `src/geo_agent/routes/jobs.py`
- Create: `tests/test_jobs_api.py`

- [ ] **Step 1: Write the failing API smoke test**

```python
from fastapi.testclient import TestClient

from geo_agent.main import app


client = TestClient(app)


def test_healthcheck_returns_ok():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_jobs_api.py::test_healthcheck_returns_ok -v`
Expected: FAIL with `ModuleNotFoundError: No module named 'geo_agent'`

- [ ] **Step 3: Write minimal project setup**

```toml
[project]
name = "geo-agent"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
  "fastapi>=0.116,<1.0",
  "uvicorn>=0.35,<1.0",
  "sqlalchemy>=2.0,<3.0",
  "pydantic>=2.11,<3.0",
  "pydantic-settings>=2.10,<3.0",
]

[project.optional-dependencies]
dev = [
  "pytest>=8.3,<9.0",
  "httpx>=0.28,<1.0",
]

[tool.pytest.ini_options]
pythonpath = ["src"]
```

```python
# src/geo_agent/main.py
from fastapi import FastAPI

app = FastAPI(title="GEO Agent MVP")


@app.get("/health")
def healthcheck() -> dict[str, str]:
    return {"status": "ok"}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/test_jobs_api.py::test_healthcheck_returns_ok -v`
Expected: PASS

- [ ] **Step 5: Add job request/response contract and route scaffold**

```python
# src/geo_agent/schemas.py
from pydantic import BaseModel, Field


class CreateJobRequest(BaseModel):
    keyword: str = Field(min_length=2)
    product_name: str = Field(min_length=2)
    target_country: str = Field(min_length=2)
    target_language: str = Field(min_length=2)
    brand_name: str | None = None
    tone: str = "professional"
    knowledge_source_paths: list[str] = []


class JobResponse(BaseModel):
    job_id: str
    status: str
```

```python
# src/geo_agent/routes/jobs.py
from uuid import uuid4

from fastapi import APIRouter

from geo_agent.schemas import CreateJobRequest, JobResponse

router = APIRouter(prefix="/jobs", tags=["jobs"])


@router.post("", response_model=JobResponse)
def create_job(_: CreateJobRequest) -> JobResponse:
    return JobResponse(job_id=str(uuid4()), status="queued")
```

```python
# src/geo_agent/main.py
from fastapi import FastAPI

from geo_agent.routes.jobs import router as jobs_router

app = FastAPI(title="GEO Agent MVP")
app.include_router(jobs_router)


@app.get("/health")
def healthcheck() -> dict[str, str]:
    return {"status": "ok"}
```

- [ ] **Step 6: Add the failing job creation test**

```python
def test_create_job_returns_queued_status():
    payload = {
        "keyword": "cnc machining parts",
        "product_name": "Precision CNC Parts",
        "target_country": "US",
        "target_language": "en",
        "knowledge_source_paths": ["storage/knowledge/cnc-faq.md"],
    }

    response = client.post("/jobs", json=payload)

    assert response.status_code == 200
    body = response.json()
    assert body["status"] == "queued"
    assert body["job_id"]
```

- [ ] **Step 7: Run the API test file**

Run: `pytest tests/test_jobs_api.py -v`
Expected: PASS

- [ ] **Step 8: Commit**

```bash
git add pyproject.toml src/geo_agent tests/test_jobs_api.py
git commit -m "feat: bootstrap geo agent api skeleton"
```

### Task 2: Persist Jobs and Generation Artifacts

**Files:**
- Modify: `src/geo_agent/config.py`
- Modify: `src/geo_agent/db.py`
- Create: `src/geo_agent/models.py`
- Modify: `src/geo_agent/routes/jobs.py`
- Modify: `src/geo_agent/schemas.py`
- Modify: `tests/test_jobs_api.py`

- [ ] **Step 1: Write the failing persistence test**

```python
def test_create_job_persists_initial_record():
    payload = {
        "keyword": "cnc machining parts",
        "product_name": "Precision CNC Parts",
        "target_country": "US",
        "target_language": "en",
        "knowledge_source_paths": [],
    }

    create_response = client.post("/jobs", json=payload)
    job_id = create_response.json()["job_id"]

    get_response = client.get(f"/jobs/{job_id}")

    assert get_response.status_code == 200
    body = get_response.json()
    assert body["keyword"] == "cnc machining parts"
    assert body["status"] == "queued"
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_jobs_api.py::test_create_job_persists_initial_record -v`
Expected: FAIL with `404 Not Found` for `GET /jobs/{job_id}`

- [ ] **Step 3: Add database settings and models**

```python
# src/geo_agent/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    database_url: str = "sqlite:///./geo_agent.db"
    model_config = SettingsConfigDict(env_prefix="GEO_AGENT_")


settings = Settings()
```

```python
# src/geo_agent/db.py
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from geo_agent.config import settings

engine = create_engine(settings.database_url, future=True)
SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False, future=True)


class Base(DeclarativeBase):
    pass
```

```python
# src/geo_agent/models.py
from sqlalchemy import JSON, String, Text
from sqlalchemy.orm import Mapped, mapped_column

from geo_agent.db import Base


class Job(Base):
    __tablename__ = "jobs"

    id: Mapped[str] = mapped_column(String, primary_key=True)
    keyword: Mapped[str] = mapped_column(String, nullable=False)
    product_name: Mapped[str] = mapped_column(String, nullable=False)
    target_country: Mapped[str] = mapped_column(String, nullable=False)
    target_language: Mapped[str] = mapped_column(String, nullable=False)
    tone: Mapped[str] = mapped_column(String, nullable=False)
    status: Mapped[str] = mapped_column(String, nullable=False)
    knowledge_source_paths: Mapped[list[str]] = mapped_column(JSON, default=list)
    article_markdown: Mapped[str | None] = mapped_column(Text)
    review_notes: Mapped[list[str] | None] = mapped_column(JSON)
```

- [ ] **Step 4: Wire app startup, create/read endpoints, and response schema**

```python
# src/geo_agent/schemas.py
from pydantic import BaseModel, Field


class CreateJobRequest(BaseModel):
    keyword: str = Field(min_length=2)
    product_name: str = Field(min_length=2)
    target_country: str = Field(min_length=2)
    target_language: str = Field(min_length=2)
    brand_name: str | None = None
    tone: str = "professional"
    knowledge_source_paths: list[str] = []


class JobResponse(BaseModel):
    job_id: str
    status: str


class JobDetailResponse(BaseModel):
    job_id: str
    keyword: str
    status: str
    article_markdown: str | None = None
    review_notes: list[str] = []
```

```python
# src/geo_agent/routes/jobs.py
from uuid import uuid4

from fastapi import APIRouter, HTTPException

from geo_agent.db import SessionLocal
from geo_agent.models import Job
from geo_agent.schemas import CreateJobRequest, JobDetailResponse, JobResponse

router = APIRouter(prefix="/jobs", tags=["jobs"])


@router.post("", response_model=JobResponse)
def create_job(payload: CreateJobRequest) -> JobResponse:
    job = Job(
        id=str(uuid4()),
        keyword=payload.keyword,
        product_name=payload.product_name,
        target_country=payload.target_country,
        target_language=payload.target_language,
        tone=payload.tone,
        status="queued",
        knowledge_source_paths=payload.knowledge_source_paths,
    )
    with SessionLocal() as session:
        session.add(job)
        session.commit()
    return JobResponse(job_id=job.id, status=job.status)


@router.get("/{job_id}", response_model=JobDetailResponse)
def get_job(job_id: str) -> JobDetailResponse:
    with SessionLocal() as session:
        job = session.get(Job, job_id)
        if job is None:
            raise HTTPException(status_code=404, detail="job not found")
        return JobDetailResponse(
            job_id=job.id,
            keyword=job.keyword,
            status=job.status,
            article_markdown=job.article_markdown,
            review_notes=job.review_notes or [],
        )
```

- [ ] **Step 5: Initialize tables at app startup**

```python
# src/geo_agent/main.py
from fastapi import FastAPI

from geo_agent.db import Base, engine
from geo_agent.routes.jobs import router as jobs_router

app = FastAPI(title="GEO Agent MVP")
Base.metadata.create_all(bind=engine)
app.include_router(jobs_router)


@app.get("/health")
def healthcheck() -> dict[str, str]:
    return {"status": "ok"}
```

- [ ] **Step 6: Run the API tests**

Run: `pytest tests/test_jobs_api.py -v`
Expected: PASS

- [ ] **Step 7: Commit**

```bash
git add src/geo_agent tests/test_jobs_api.py
git commit -m "feat: persist geo generation jobs"
```

### Task 3: Build Keyword Understanding and Planning

**Files:**
- Create: `src/geo_agent/services/__init__.py`
- Create: `src/geo_agent/services/keyword_understanding.py`
- Create: `src/geo_agent/services/content_planning.py`
- Create: `tests/test_keyword_understanding.py`
- Create: `tests/test_content_planning.py`

- [ ] **Step 1: Write the failing keyword understanding tests**

```python
from geo_agent.services.keyword_understanding import analyze_keyword


def test_analyze_keyword_detects_commercial_intent():
    result = analyze_keyword("best cnc machining service for aluminum parts")
    assert result.intent == "commercial"
    assert result.funnel_stage == "consideration"


def test_analyze_keyword_detects_question_intent():
    result = analyze_keyword("what is cnc machining tolerance")
    assert result.intent == "informational"
    assert result.primary_angle == "question"
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_keyword_understanding.py -v`
Expected: FAIL with `ModuleNotFoundError` for `geo_agent.services.keyword_understanding`

- [ ] **Step 3: Implement minimal keyword analysis**

```python
# src/geo_agent/services/keyword_understanding.py
from dataclasses import dataclass


@dataclass(frozen=True)
class KeywordAnalysis:
    intent: str
    funnel_stage: str
    primary_angle: str
    risk_level: str


def analyze_keyword(keyword: str) -> KeywordAnalysis:
    normalized = keyword.lower()
    if normalized.startswith(("what ", "how ", "why ")) or normalized.startswith(("what is", "how to")):
        return KeywordAnalysis("informational", "awareness", "question", "low")
    if any(token in normalized for token in ["best", "service", "supplier", "company"]):
        return KeywordAnalysis("commercial", "consideration", "comparison", "medium")
    return KeywordAnalysis("informational", "consideration", "feature", "low")
```

- [ ] **Step 4: Write the failing content planning test**

```python
from geo_agent.services.content_planning import build_article_plan
from geo_agent.services.keyword_understanding import KeywordAnalysis


def test_build_article_plan_creates_geo_outline():
    analysis = KeywordAnalysis(
        intent="commercial",
        funnel_stage="consideration",
        primary_angle="comparison",
        risk_level="medium",
    )

    plan = build_article_plan(
        keyword="best cnc machining service for aluminum parts",
        product_name="Precision CNC Parts",
        analysis=analysis,
    )

    assert "direct_answer" in plan.sections
    assert "faq" in plan.sections
    assert plan.article_type == "commercial_comparison"
```

- [ ] **Step 5: Run planning test to verify it fails**

Run: `pytest tests/test_content_planning.py -v`
Expected: FAIL with `ModuleNotFoundError` for `geo_agent.services.content_planning`

- [ ] **Step 6: Implement minimal article planning**

```python
# src/geo_agent/services/content_planning.py
from dataclasses import dataclass

from geo_agent.services.keyword_understanding import KeywordAnalysis


@dataclass(frozen=True)
class ArticlePlan:
    article_type: str
    title_hint: str
    sections: list[str]


def build_article_plan(keyword: str, product_name: str, analysis: KeywordAnalysis) -> ArticlePlan:
    article_type = "commercial_comparison" if analysis.intent == "commercial" else "problem_solving"
    title_hint = f"{keyword.title()} | {product_name}"
    sections = ["direct_answer", "problem_context", "evidence", "faq", "cta"]
    return ArticlePlan(article_type=article_type, title_hint=title_hint, sections=sections)
```

- [ ] **Step 7: Run understanding and planning tests**

Run: `pytest tests/test_keyword_understanding.py tests/test_content_planning.py -v`
Expected: PASS

- [ ] **Step 8: Commit**

```bash
git add src/geo_agent/services tests/test_keyword_understanding.py tests/test_content_planning.py
git commit -m "feat: add keyword understanding and article planning"
```

### Task 4: Register Knowledge Sources and Retrieve Evidence

**Files:**
- Create: `src/geo_agent/services/knowledge_base.py`
- Create: `src/geo_agent/services/evidence_retrieval.py`
- Create: `tests/test_knowledge_base.py`
- Create: `storage/knowledge/cnc-faq.md`

- [ ] **Step 1: Write the failing knowledge registration test**

```python
from pathlib import Path

from geo_agent.services.knowledge_base import load_knowledge_sources


def test_load_knowledge_sources_reads_markdown_chunks(tmp_path: Path):
    source = tmp_path / "cnc.md"
    source.write_text("# CNC\nTolerance matters for aerospace parts.", encoding="utf-8")

    chunks = load_knowledge_sources([str(source)])

    assert len(chunks) == 1
    assert "Tolerance matters" in chunks[0].content
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_knowledge_base.py::test_load_knowledge_sources_reads_markdown_chunks -v`
Expected: FAIL with `ModuleNotFoundError` for `geo_agent.services.knowledge_base`

- [ ] **Step 3: Implement source loading**

```python
# src/geo_agent/services/knowledge_base.py
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class KnowledgeChunk:
    source_path: str
    content: str


def load_knowledge_sources(paths: list[str]) -> list[KnowledgeChunk]:
    chunks: list[KnowledgeChunk] = []
    for path in paths:
        content = Path(path).read_text(encoding="utf-8")
        chunks.append(KnowledgeChunk(source_path=path, content=content))
    return chunks
```

- [ ] **Step 4: Write the failing evidence retrieval test**

```python
from geo_agent.services.evidence_retrieval import select_relevant_evidence
from geo_agent.services.knowledge_base import KnowledgeChunk


def test_select_relevant_evidence_filters_for_keyword_terms():
    chunks = [
        KnowledgeChunk(source_path="a.md", content="Aluminum CNC parts require tight tolerance."),
        KnowledgeChunk(source_path="b.md", content="Wood furniture polishing guide."),
    ]

    selected = select_relevant_evidence("aluminum cnc parts", chunks)

    assert len(selected) == 1
    assert selected[0].source_path == "a.md"
```

- [ ] **Step 5: Run the retrieval test to verify it fails**

Run: `pytest tests/test_knowledge_base.py::test_select_relevant_evidence_filters_for_keyword_terms -v`
Expected: FAIL with `ModuleNotFoundError` for `geo_agent.services.evidence_retrieval`

- [ ] **Step 6: Implement evidence retrieval and sample knowledge**

```python
# src/geo_agent/services/evidence_retrieval.py
from geo_agent.services.knowledge_base import KnowledgeChunk


def select_relevant_evidence(keyword: str, chunks: list[KnowledgeChunk]) -> list[KnowledgeChunk]:
    terms = {term for term in keyword.lower().split() if len(term) > 2}
    return [chunk for chunk in chunks if any(term in chunk.content.lower() for term in terms)]
```

```markdown
<!-- storage/knowledge/cnc-faq.md -->
# Precision CNC FAQ

- Aluminum CNC parts usually require tolerance references, material notes, and process selection guidance.
- Buyers often compare lead time, surface finish, and quality inspection standards.
```

- [ ] **Step 7: Run the knowledge tests**

Run: `pytest tests/test_knowledge_base.py -v`
Expected: PASS

- [ ] **Step 8: Commit**

```bash
git add src/geo_agent/services/knowledge_base.py src/geo_agent/services/evidence_retrieval.py tests/test_knowledge_base.py storage/knowledge/cnc-faq.md
git commit -m "feat: add local knowledge loading and evidence retrieval"
```

### Task 5: Generate Article Drafts and Review Notes

**Files:**
- Create: `src/geo_agent/llm/__init__.py`
- Create: `src/geo_agent/llm/base.py`
- Create: `src/geo_agent/llm/mock.py`
- Create: `src/geo_agent/services/article_generation.py`
- Create: `src/geo_agent/services/review.py`
- Create: `tests/test_review.py`

- [ ] **Step 1: Write the failing review test**

```python
from geo_agent.services.review import review_article


def test_review_article_flags_missing_evidence():
    notes = review_article(
        article_markdown="## Answer\nWe are the best supplier in the world.",
        evidence_lines=[],
    )

    assert "Missing evidence-backed support" in notes
    assert "Avoid unsupported absolute claim: best" in notes
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_review.py -v`
Expected: FAIL with `ModuleNotFoundError` for `geo_agent.services.review`

- [ ] **Step 3: Implement article generation and review rules**

```python
# src/geo_agent/llm/base.py
from typing import Protocol


class LLMClient(Protocol):
    def complete(self, prompt: str) -> str: ...
```

```python
# src/geo_agent/llm/mock.py
from geo_agent.llm.base import LLMClient


class MockLLM(LLMClient):
    def complete(self, prompt: str) -> str:
        return prompt
```

```python
# src/geo_agent/services/article_generation.py
from geo_agent.services.content_planning import ArticlePlan
from geo_agent.services.knowledge_base import KnowledgeChunk


def generate_article_markdown(keyword: str, plan: ArticlePlan, evidence: list[KnowledgeChunk]) -> str:
    bullet_lines = "\n".join(f"- Source: {chunk.content[:100]}" for chunk in evidence[:3])
    return (
        f"# {plan.title_hint}\n\n"
        f"## Direct Answer\n{keyword} buyers need a concise answer supported by source material.\n\n"
        f"## Evidence\n{bullet_lines}\n\n"
        f"## FAQ\n- What should buyers compare first?\n"
    )
```

```python
# src/geo_agent/services/review.py
def review_article(article_markdown: str, evidence_lines: list[str]) -> list[str]:
    notes: list[str] = []
    if not evidence_lines:
        notes.append("Missing evidence-backed support")
    if "best" in article_markdown.lower():
        notes.append("Avoid unsupported absolute claim: best")
    if "## FAQ" not in article_markdown:
        notes.append("Missing FAQ section for GEO readability")
    return notes
```

- [ ] **Step 4: Run the review test**

Run: `pytest tests/test_review.py -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/geo_agent/llm src/geo_agent/services/article_generation.py src/geo_agent/services/review.py tests/test_review.py
git commit -m "feat: generate article drafts with basic review rules"
```

### Task 6: Orchestrate the End-to-End Pipeline in the Jobs API

**Files:**
- Create: `src/geo_agent/services/pipeline.py`
- Modify: `src/geo_agent/routes/jobs.py`
- Modify: `tests/test_jobs_api.py`

- [ ] **Step 1: Write the failing end-to-end job execution test**

```python
def test_create_job_runs_pipeline_and_returns_article():
    payload = {
        "keyword": "best cnc machining service for aluminum parts",
        "product_name": "Precision CNC Parts",
        "target_country": "US",
        "target_language": "en",
        "knowledge_source_paths": ["storage/knowledge/cnc-faq.md"],
    }

    create_response = client.post("/jobs", json=payload)
    job_id = create_response.json()["job_id"]

    get_response = client.get(f"/jobs/{job_id}")
    body = get_response.json()

    assert body["status"] == "completed"
    assert "# Best Cnc Machining Service For Aluminum Parts | Precision CNC Parts" in body["article_markdown"]
    assert isinstance(body["review_notes"], list)
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/test_jobs_api.py::test_create_job_runs_pipeline_and_returns_article -v`
Expected: FAIL because status is still `queued`

- [ ] **Step 3: Implement the pipeline**

```python
# src/geo_agent/services/pipeline.py
from dataclasses import dataclass

from geo_agent.services.article_generation import generate_article_markdown
from geo_agent.services.content_planning import build_article_plan
from geo_agent.services.evidence_retrieval import select_relevant_evidence
from geo_agent.services.keyword_understanding import analyze_keyword
from geo_agent.services.knowledge_base import load_knowledge_sources
from geo_agent.services.review import review_article


@dataclass(frozen=True)
class PipelineResult:
    article_markdown: str
    review_notes: list[str]


def run_pipeline(keyword: str, product_name: str, source_paths: list[str]) -> PipelineResult:
    analysis = analyze_keyword(keyword)
    plan = build_article_plan(keyword=keyword, product_name=product_name, analysis=analysis)
    chunks = load_knowledge_sources(source_paths)
    evidence = select_relevant_evidence(keyword, chunks)
    article = generate_article_markdown(keyword, plan, evidence)
    notes = review_article(article, [chunk.content for chunk in evidence])
    return PipelineResult(article_markdown=article, review_notes=notes)
```

- [ ] **Step 4: Wire the pipeline into job creation**

```python
# src/geo_agent/routes/jobs.py
from uuid import uuid4

from fastapi import APIRouter, HTTPException

from geo_agent.db import SessionLocal
from geo_agent.models import Job
from geo_agent.schemas import CreateJobRequest, JobDetailResponse, JobResponse
from geo_agent.services.pipeline import run_pipeline

router = APIRouter(prefix="/jobs", tags=["jobs"])


@router.post("", response_model=JobResponse)
def create_job(payload: CreateJobRequest) -> JobResponse:
    pipeline_result = run_pipeline(
        keyword=payload.keyword,
        product_name=payload.product_name,
        source_paths=payload.knowledge_source_paths,
    )
    job = Job(
        id=str(uuid4()),
        keyword=payload.keyword,
        product_name=payload.product_name,
        target_country=payload.target_country,
        target_language=payload.target_language,
        tone=payload.tone,
        status="completed",
        knowledge_source_paths=payload.knowledge_source_paths,
        article_markdown=pipeline_result.article_markdown,
        review_notes=pipeline_result.review_notes,
    )
    with SessionLocal() as session:
        session.add(job)
        session.commit()
    return JobResponse(job_id=job.id, status=job.status)
```

- [ ] **Step 5: Run the full test suite**

Run: `pytest -v`
Expected: PASS

- [ ] **Step 6: Commit**

```bash
git add src/geo_agent/routes/jobs.py src/geo_agent/services/pipeline.py tests/test_jobs_api.py
git commit -m "feat: wire geo article pipeline into jobs api"
```

### Task 7: Add MVP Documentation and Example Usage

**Files:**
- Create: `docs/geo-agent-mvp.md`
- Modify: `README.md`

- [ ] **Step 1: Add usage documentation**

```markdown
# GEO Agent MVP

## What this MVP does

- Accepts keyword, product context, and local knowledge source paths
- Classifies keyword intent
- Builds a GEO-friendly article plan
- Pulls supporting evidence from local files
- Generates a draft article plus review notes

## Run locally

```bash
pip install -e .[dev]
uvicorn geo_agent.main:app --reload
```

## Example request

```bash
curl -X POST http://127.0.0.1:8000/jobs \
  -H "Content-Type: application/json" \
  -d "{\"keyword\":\"best cnc machining service for aluminum parts\",\"product_name\":\"Precision CNC Parts\",\"target_country\":\"US\",\"target_language\":\"en\",\"knowledge_source_paths\":[\"storage/knowledge/cnc-faq.md\"]}"
```
```

- [ ] **Step 2: Link the MVP guide from the repo README**

```markdown
# test

Project scratchpad for the GEO article agent MVP.

- Plan: `docs/superpowers/plans/2026-04-08-geo-agent-mvp.md`
- Usage guide: `docs/geo-agent-mvp.md`
```

- [ ] **Step 3: Commit**

```bash
git add README.md docs/geo-agent-mvp.md
git commit -m "docs: add geo agent mvp usage guide"
```

## Self-Review

**Spec coverage:** This plan covers the MVP pipeline you asked for: keyword input,外挂知识库接入、关键词理解、内容规划、证据抽取、文章生成、审核、结果输出。 It does not cover front-end UI, scheduling, publishing, ranking analytics, or multi-user SaaS features.

**Placeholder scan:** No `TODO`, `TBD`, or “implement later” placeholders are left in the task steps. Each code step includes concrete code and each verification step includes explicit commands.

**Type consistency:** `CreateJobRequest`, `JobResponse`, `JobDetailResponse`, `KeywordAnalysis`, `ArticlePlan`, `KnowledgeChunk`, and `PipelineResult` are named consistently across tasks, and later tasks reuse the exact earlier names.

Plan complete and saved to `docs/superpowers/plans/2026-04-08-geo-agent-mvp.md`. Two execution options:

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?**
