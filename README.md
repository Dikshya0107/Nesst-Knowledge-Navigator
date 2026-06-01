# NESsT Knowledge Navigator 🧭

## Overview

NESsT Knowledge Navigator is an AI-powered knowledge management platform developed during the Cisco AI Hackathon to help organizations unlock insights from decades of accumulated documentation.

The platform enables users to search, analyze, and summarize information across thousands of documents using Retrieval-Augmented Generation (RAG), significantly reducing the time required to locate critical information.

---

# Problem Statement

NESsT, a global nonprofit organization, has accumulated over three decades of institutional knowledge across thousands of documents, reports, and research materials.

This created several challenges:

* Knowledge was distributed across multiple folders and repositories.
* Locating relevant information often required manual searching through numerous documents.
* Important organizational knowledge remained locked inside PDFs and reports.
* Employees spent significant time searching for answers instead of acting on information.
* Cross-document analysis and summarization were difficult and time-consuming.

The goal was to build an intelligent system capable of transforming large volumes of unstructured content into a searchable and accessible knowledge base.

---

# Solution Overview

NESsT Knowledge Navigator leverages Retrieval-Augmented Generation (RAG) to provide accurate and context-aware responses from organizational documents.

Key capabilities include:

* Semantic document search
* Natural language question answering
* Multi-document analysis
* Executive-level summarization
* Template-based report generation
* Role-based access control (Admin/Public)
* Automated document ingestion and indexing

Users can upload documents, ask questions in natural language, and receive synthesized responses generated from relevant source material.

---

# Architecture

```text
                    ┌─────────────────┐
                    │     Frontend    │
                    │ HTML/CSS/JS UI  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ FastAPI Backend │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼

 ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
 │ Document       │  │ ChromaDB       │  │ Authentication │
 │ Ingestion      │  │ Vector Store   │  │ & Access Mgmt  │
 └────────────────┘  └────────────────┘  └────────────────┘
          │
          ▼
 ┌────────────────┐
 │ Chunking &     │
 │ Embedding      │
 └────────────────┘
          │
          ▼
 ┌────────────────┐
 │ RAG Pipeline   │
 │ Retrieve +     │
 │ Generate       │
 └────────────────┘
          │
          ▼
 ┌────────────────┐
 │ GPT-5.2 via    │
 │ Cisco Playground│
 └────────────────┘
```

### Workflow

1. Documents are uploaded and processed.
2. Content is parsed, cleaned, and chunked.
3. Embeddings are generated and stored in ChromaDB.
4. User queries are converted into embeddings.
5. Relevant document chunks are retrieved using semantic search.
6. Retrieved context is provided to the LLM.
7. The LLM generates grounded responses based on retrieved content.

---

# Technology Stack

| Layer           | Technology                           |
| --------------- | ------------------------------------ |
| Frontend        | HTML, CSS, JavaScript                |
| Backend         | Python, FastAPI                      |
| Vector Database | ChromaDB                             |
| Embeddings      | text-embedding-3-large               |
| LLM             | GPT-5.2 via Cisco Playground         |
| AI Architecture | Retrieval-Augmented Generation (RAG) |
| Authentication  | Role-Based Access Control            |

---

# My Contributions

As part of a 4-member engineering team, I contributed across the full software development lifecycle, including design, implementation, and testing.

### Full-Stack Development

* Developed frontend features for document interaction and search workflows.
* Built backend APIs and integrated AI services.
* Implemented document upload and processing workflows.
* Integrated vector search capabilities using ChromaDB.
* Supported role-based access management for different user types.

### Testing & Quality Assurance

* Performed end-to-end validation of document ingestion and retrieval workflows.
* Tested query accuracy across multiple document types.
* Validated system behavior under various user scenarios.
* Identified and resolved integration issues across frontend, backend, and AI components.

### Collaboration

* Worked closely with team members to integrate frontend, backend, database, and AI components.
* Participated in architecture discussions and technical decision-making.
* Contributed to feature prioritization during hackathon development.

---

# Engineering Challenges & Tradeoffs

### Challenge: Large Unstructured Knowledge Base

Thousands of documents contained valuable information but lacked a unified structure.

**Approach:**
Implemented automated ingestion, chunking, and semantic indexing to enable efficient retrieval.

---

### Challenge: Retrieval Accuracy

Returning irrelevant document sections could negatively impact answer quality.

**Approach:**
Used embedding-based semantic search and metadata filtering to improve relevance.

---

### Challenge: Context Window Limitations

Large documents often exceeded LLM context limits.

**Approach:**
Implemented document chunking and retrieval strategies to provide only the most relevant context.

---

### Challenge: Balancing Speed and Quality

Increasing retrieval depth improved answer quality but increased response time.

**Tradeoff:**
Optimized retrieval parameters to balance responsiveness and accuracy.

---

# Impact

The solution demonstrated how AI can transform organizational knowledge management by:

* Reducing information discovery time from hours to seconds.
* Enabling natural language access to decades of institutional knowledge.
* Improving accessibility of information stored across thousands of documents.
* Supporting faster decision-making through summarization and analysis.
* Eliminating the need for extensive manual document review.
* Providing a scalable foundation for enterprise knowledge management.

---

# Key Learnings

This project strengthened my understanding of:

* Retrieval-Augmented Generation (RAG) systems
* Vector databases and semantic search
* LLM integration in production workflows
* Prompt engineering and response grounding
* Full-stack application development
* API design using FastAPI
* AI system evaluation and testing
* Cross-functional collaboration in rapid development environments

---

## Disclaimer

This repository is intended to showcase the architecture, engineering approach, technical decisions, and lessons learned from the project.

To respect confidentiality and intellectual property requirements, proprietary source code, internal datasets, credentials, and organization-specific implementation details are not included.
