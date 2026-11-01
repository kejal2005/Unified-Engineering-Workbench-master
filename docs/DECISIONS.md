# Architecture and Design Decisions

## Decision Log

### D1: Source Directory Organization
- **Context**: The assignment guideline specifies a root `src/` directory containing all project source components, while initial draft trees isolated `frontend/` and `backend/` at root level.
- **Decision**: Consolidate source under `src/frontend`, `src/backend`, and `src/launcher`.
- **Rationale**: Eliminates root clutter, satisfies grading evaluation criteria, and ensures single root source hierarchy for static inspection tools.

### D2: Editor Configuration Location
- **Context**: An `editor-config/` directory was suggested, but Visual Studio Code and Antigravity IDE natively look for configuration in `.vscode/`.
- **Decision**: Maintain functional `launch.json` and `tasks.json` in `.vscode/`, keeping `editor-config/README.md` strictly as documentation for environment prerequisites.
- **Rationale**: Ensures out-of-the-box run capability upon cloning or opening in supported IDEs without manual configuration copying.

### D3: Vendored Third-Party Dependencies
- **Context**: Submission evaluation requires zero external network downloads or package manager restores on test machines.
- **Decision**: Place header-only dependencies (`cpp-httplib`, `nlohmann/json`) and `GoogleTest` directly in `third_party/`, cataloged in `THIRD_PARTY.md`.
- **Rationale**: Enables deterministic offline CMake compilation without Conan, vcpkg, or CPM network calls.

### D4: Linear Solver Selection for 1D Thermal FEA
- **Context**: Third-party dense or sparse matrix libraries (e.g., Eigen) could be linked.
- **Decision**: Implement a specialized tridiagonal matrix solver (Thomas algorithm). Use a dense Gaussian elimination routine solely within test fixtures for cross-verification.
- **Rationale**: The 1D linear mesh yields a strictly tridiagonal, diagonally dominant system. The Thomas algorithm achieves $O(N)$ computational complexity and minimal memory footprint with zero external dependencies.

### D5: Handwritten Derivation Boundary
- **Context**: Backpropagation requirement calls for a 2-page handwritten derivation.
- **Decision**: Automated and AI tooling will only generate and verify numeric validation check sheets. The mathematical derivation will be handwritten, defended, and scanned manually.
- **Rationale**: Preserves academic and assignment evaluation integrity.

### D6: AI Usage and Challenge Validation
- **Context**: Evaluation requires documentation of authentic AI collaboration, including handling incorrect suggestions or structured challenge queries.
- **Decision**: Maintain strict audit logs in `docs/AI_USE_LOG.md` tracking actual tool prompts, outputs, manual corrections, and formal challenge test outcomes.
- **Rationale**: Demonstrates rigorous engineering oversight and verifiable verification.

### D7: HTTP and Transport Selection
- **Context**: IPC mechanism between frontend and backend could use WebSockets, gRPC, Named Pipes, or HTTP.
- **Decision**: Select loopback HTTP/REST using `cpp-httplib` with polling for long jobs.
- **Rationale**: Simplifies connection recovery, eliminates binary protocol dependencies, supports direct static asset serving, and avoids firewall issues associated with non-standard protocols.

