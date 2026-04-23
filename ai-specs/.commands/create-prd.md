# Role

You are an expert Product Manager and Business Analyst with extensive experience creating Product Requirements Documents (PRDs) for software products.

# Project Context File

$ARGUMENTS

# Goal

Generate a comprehensive Product Requirements Document (PRD) that serves as the foundational reference for the entire product development lifecycle, based on the project context provided in the input file.

# Process and rules

1. Adopt the role of `.claude/agents/product-manager-ba.md`
2. Read and analyze the project context file provided in `$ARGUMENTS`. This file should contain the project description, goals, target users, or any existing product documentation that serves as input for the PRD.
3. If the product already has existing documentation in the workspace (e.g., previous PRDs, user research, market analysis), review them first to ensure consistency and avoid duplication.
4. Follow a structured, iterative approach:
   - Start by understanding the problem space and target users
   - Define the product vision and objectives
   - Identify key features and use cases
   - Design the data model
   - Specify functional and non-functional requirements
5. Apply product management best practices to ensure the PRD is actionable and developer-ready.
6. All content must be written in English, following the project's documentation standards found in `/ai-specs/specs/documentation-standards.mdc`.
7. Do not write code; provide only the PRD document in the output format defined below.

# Output format

Markdown document at the path `ai-specs/specs/PRD.md` containing the complete Product Requirements Document.
Follow this template:

## PRD Template Structure

### 1. **Header**
- Title: `# Product Requirements Document: [Product Name]`
- Version, date, author (AI-assisted), status (Draft/Review/Approved)

### 2. **Executive Summary**
- Brief description of the product (2-3 paragraphs)
- Problem statement: What problem does this product solve?
- Target audience: Who is this product for?

### 3. **Product Vision & Objectives**
- Product vision statement
- Business objectives (measurable goals)
- Success metrics (KPIs)
- Alignment with company strategy (if applicable)

### 4. **Target Users & Personas**
- User persona definitions (at least 2-3 personas)
  - Name, role, demographics
  - Goals and motivations
  - Pain points and frustrations
  - Current alternatives they use
- User segmentation

### 5. **Use Cases & User Stories**
- Primary use cases with detailed scenarios
  - Actors involved
  - Preconditions
  - Main flow (step-by-step)
  - Postconditions
  - Alternative flows
  - Exception flows
- User Stories in standard format:
  - `As a [persona], I want to [action], so that [benefit]`
  - Acceptance criteria for each story
  - Priority (MoSCoW: Must have, Should have, Could have, Won't have)

### 6. **Key Features & Functional Requirements**
- Feature list organized by priority
- For each feature:
  - Feature ID and name
  - Description
  - User stories linked
  - Acceptance criteria
  - Dependencies
  - Priority and estimated complexity

### 7. **Data Model**
- Entity-Relationship diagram description
- Key entities and their attributes
- Relationships between entities
- Data validation rules
- Data retention and privacy considerations

### 8. **System Architecture (High-Level)**
- Architecture overview (components and their interactions)
- Technology stack recommendations
- Integration points with external systems
- Scalability considerations

### 9. **API Specification (High-Level)**
- Key endpoints organized by resource
- For each endpoint:
  - HTTP method and URL pattern
  - Request/response structure summary
  - Authentication requirements

### 10. **Non-Functional Requirements**
- Performance requirements (response times, throughput)
- Security requirements (authentication, authorization, data protection)
- Scalability requirements
- Availability and reliability targets
- Accessibility requirements (WCAG compliance level)
- Internationalization/Localization needs

### 11. **UX/UI Guidelines**
- Design principles
- Key screens/wireframe descriptions
- Navigation flow
- Responsive design considerations
- Branding and style requirements

### 12. **Product Backlog**
- Prioritized backlog of user stories
- Prioritization methodology used (RICE, MoSCoW, Weighted Scoring, etc.)
- Release phases / MVP definition
- Dependencies between stories

### 13. **Risks & Assumptions**
- Known risks with mitigation strategies
- Key assumptions that need validation
- Open questions

### 14. **Glossary**
- Domain-specific terms and definitions

### 15. **Appendix**
- References to related documents
- Market research summaries
- Competitive analysis highlights
