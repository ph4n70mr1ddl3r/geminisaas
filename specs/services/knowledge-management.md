# Knowledge Management Service Specification

## Overview

The Knowledge Management service manages the creation, approval, and distribution of information articles, troubleshooting steps, and company policies. This aligns with Oracle Fusion Cloud Knowledge Management.

## Data Model (SQLite)

- `knowledge_articles`: id (UUID), title, body (HTML/Markdown), summary, category_id, language, status (DRAFT/APPROVED/EXPIRED), version.
- `article_categories`: id (UUID), name, parent_id, status.
- `article_approvals`: id (UUID), article_id, approver_id, status (PENDING/APPROVED/REJECTED), comments, date.
- `article_feedback`: id (UUID), article_id, user_id, rating (1-5), comments, date.
- `knowledge_search_logs`: id (UUID), user_id, search_term, clicked_article_id, timestamp.
- `article_translations`: id (UUID), article_id, language, localized_title, localized_body.

## API Endpoints (Axum/gRPC)

- `GET /knowledge/articles/search`: Search for articles using keywords or categories.
- `POST /knowledge/articles`: Submit a new knowledge base article for review.
- `POST /knowledge/articles/{id}/approve`: Record an approval step.
- `GET /knowledge/articles/{id}`: View article content and metadata.
- `POST /knowledge/feedback`: Provide feedback on article helpfulness.
- `GET /knowledge/analytics/popular`: Retrieve most viewed or highest rated articles.

## Inter-service Interaction

- Queries `CRM Service` for customer service context to recommend relevant articles.
- Emits `Article_Created` event to `Marketing` to potentially include in customer newsletters.
- Queries `Identity Service` for author and approver roles.
- Emits `New_Feedback_Received` to content authors for article improvement.

## Key Logic

- **Semantic Search:** Using natural language processing (NLP) to find relevant articles even if keywords don't match exactly.
- **AI-Generated Summarization:** Automatically creating a concise summary of long articles for quick scanning.
- **Version Control:** Maintaining a history of changes to ensure only the most recent approved version is visible to customers.
- **Content Expiration:** Automatically flagging articles for review after a certain period (e.g., 6 months).
- **Localized Delivery:** Serving the article in the user's preferred language based on available translations.
