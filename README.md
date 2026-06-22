(dsstar) PS D:\python\기타과제\20260622_DS_STAR> uv sync
warning: The `tool.uv.dev-dependencies` field (used in `pyproject.toml`) is deprecated and will be removed in a future release; use `dependency-groups.dev` instead
warning: `VIRTUAL_ENV=dsstar` does not match the project environment path `.venv` and will be ignored; use `--active` to target the active environment instead
Resolved 49 packages in 2ms
  × Failed to download `grpcio==1.76.0`
  ├─▶ Request failed after 3 retries in 4.8s                                                                                                                                                             
  ├─▶ Failed to fetch: `https://files.pythonhosted.org/packages/60/9c/5c359c8d4c9176cfa3c61ecd4efe5affe1f38d9bae81e81ac7186b4c9cc8/grpcio-1.76.0-cp311-cp311-win_amd64.whl`
  ├─▶ error sending request for url (https://files.pythonhosted.org/packages/60/9c/5c359c8d4c9176cfa3c61ecd4efe5affe1f38d9bae81e81ac7186b4c9cc8/grpcio-1.76.0-cp311-cp311-win_amd64.whl)
  ├─▶ client error (Connect)
  ╰─▶ invalid peer certificate: UnknownIssuer

hint: `grpcio` (v1.76.0) was included because `ds-star` (v0.1.0) depends on `google-generativeai` (v0.8.5) which depends on `google-api-core[grpc]` (v2.25.2) which depends on `grpcio`
