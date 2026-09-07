# Fresh source deployment verification

Completed across 7-8 September 2026, Australia/Melbourne.

## Isolation and procedure

The reviewed source ZIP was extracted into a new folder. Before setup, the folder contained no `.env`, `.env.docker`, `node_modules` or `.git`. No host Node installation was used for application setup or tests. The Docker-only setup command generated new private configuration. A separate Compose project named `libswap-freshcheck` used port 3011 and a newly created named MongoDB volume.

The app image was built with `docker compose build --pull --no-cache`. Tests then ran inside the new application container. Both containers were removed and recreated without deleting the named volume to verify persistence.

## Results

| Check | Result |
| --- | --- |
| Docker-only generation of new configuration | Passed |
| Pull base image and build without application cache | Passed |
| New app and database containers reach healthy state | Passed |
| Demo accounts and books created from empty database | Passed |
| Registration, login, circulation, validation and concurrency integration test | Passed |
| Student identity and all application pages | Passed |
| Borrow and reserve before container recreation | Passed |
| Recreate both containers, retaining only the named database volume | Passed |
| Loan and reservation survive recreation | Passed |
| Queue priority, reserved borrowing and final return | Passed |

The full non-secret command output is in `fresh-environment.log` next to this record.

## Scope and remaining verification

This was a fresh source-package deployment using the existing Docker Desktop installation on the same Mac. Docker's existing Linux VM and image layers were not replaced. This is not evidence of a separate physical machine or independently provisioned VM, nor of a public GitHub clone.

`.github/workflows/docker-smoke.yml` is ready to run on a fresh GitHub-hosted Ubuntu runner after publication. A successful Actions run and its URL must be checked before claiming that verification. The assignment strongly recommends fresh-environment testing; the strongest remaining check is to clone the final public repository on an independent machine or runner and follow only README.md.
