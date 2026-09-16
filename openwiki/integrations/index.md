# Files

- [GitHub Actions and CI/CD](github-actions.md) - Repository automation separates untrusted pull-request checks from scheduled and maintainer-controlled workflows that use credentials or write state. It includes changed-document version validation and weekly synchronization of mirrored upstream requirements.
- [Mintlify Integration](mintlify.md) - Mintlify renders the generated LangChain documentation tree and uses docs.json as its renderer-facing navigation and redirect contract. This page covers the generated-tree boundary, OpenAPI ownership, local validation, previews, and production publication.
- [NPM Snippet Components](npm-snippets.md) - How the builder overlays sandbox components from @langchain/docs-sandbox into generated documentation, how MDX pages consume them, and how to validate the resulting output.
- [Reference Documentation Integration](reference-docs.md) - Defines the boundary between semantic SDK links, externally operated API reference sites, and OpenAPI inputs that Mintlify turns into LangSmith endpoint documentation at deployment. Covers refresh automation and checks designed for generated routes.
