Date: 2026-04-27

Summary:
Set an explicit Docker Compose project name in the repository root so Docker resources stop using the default folder-derived name `raptor` and use `raptor-chatbot` instead.

Files created/modified:
- `e:/raptor/docker-compose.yml` — added top-level `name: raptor-chatbot` to control Compose project naming.
- `e:/raptor/.claude/contexts/docker-compose-project-name-2026-04-27.md` — recorded the change context for future reference.

Decisions made:
- Used the top-level Compose `name` field instead of adding `container_name` per service, because the request was about the instance/project name and this keeps service discovery and Compose-managed naming intact.
- Avoided changing service names, image names, or documentation because they are not required for the Docker project rename.

Known issues or next steps:
- Existing resources created under the old project name (`raptor_*`) will remain until they are recreated or cleaned up.
- Re-run `docker compose up -d` from `e:/raptor` to create containers, networks, and volumes under the new `raptor-chatbot` prefix.
