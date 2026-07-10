# 002 — SPA frontend architecture

- Status: accepted
- Date: 2026-07-07

## Context

We are building a SPA frontend for the two-sided marketplace application, and needed to consider the constraint around being able to deploy a static site to S3 and CloudFront.

A meta framework, such as Tanstack Start was considered, however we rejected this idea in favor of a more minimal approach as these are largely built with SSR/SSG and server-components in mind, which requires additional infrastructure and complexity to support.

## Decision

What we chose, stated plainly in the active voice ("We will...").

We chose to build the frontend using the following high-level stack:

- Vite for the build system
- React as a UI library
- CSS Modules (no plugins, natively supported)
- TypeScript for type safety

This excludes any additional libraries or tools, such as Prettier, ESLint as this will be covered in a separate ADR, as it encompases more than just the frontend.

## Consequences

By rolling our frontend as a basic Vite project, and without the help of a meta framework, we cannot take advantage of server-rendered HTML for SEO benefits and improved initial page load times. Additionally, we must include all code in our client-side bundle, which can increase the initial bundle size and impact performance, as there is no way to split the code into a separate server-side bundle.

The trade-off here is that we lose the benefits of a meta framework, but gain a simpler execution environment, where the following is tied to the browser:

- Data fetching / request model
- Rendering
- State Management
- Routing

There is no need for maintaining a separate frontend server-side execution environment, allowing us to simply ship static assets and store them in S3 and serve them via CloudFront.
