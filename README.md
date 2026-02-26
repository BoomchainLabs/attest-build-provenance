# BoomchainLabs Attest Build Provenance

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Attest%20Build%20Provenance-blue?logo=github)](https://github.com/marketplace/actions/attest-build-provenance)  
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)

Generate signed build provenance attestations for Docker images and other workflow artifacts. Ensure your builds are **secure, verifiable, and fully traceable**.

---

## Features

- Attest **Docker images** and push attestations to container registries.  
- Attest **arbitrary build artifacts** (binaries, files, etc.).  
- Supports **SLSA-compliant predicates**.  
- Attach attestation summaries to workflow runs.  
- Create **GitHub storage records** for artifacts.  
- Fully compatible with **GitHub Actions Marketplace**.  

---

## Usage

### Attest a Docker Image

```yaml
jobs:
  attest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t ghcr.io/${{ github.repository }}/my-app:latest .

      - name: Push Docker image
        run: docker push ghcr.io/${{ github.repository }}/my-app:latest

      - name: Get image digest
        id: digest
        run: |
          DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' ghcr.io/${{ github.repository }}/my-app:latest | cut -d'@' -f2)
          echo "digest=sha256:$DIGEST" >> $GITHUB_OUTPUT

      - name: Attest Docker image
        uses: BoomchainLabs/attest-build-provenance@v1.0.0
        with:
          subject-name: ghcr.io/${{ github.repository }}/my-app
          subject-digest: ${{ steps.digest.outputs.digest }}
          predicate-type: https://slsa.dev/provenance/v0.2
          predicate: '{"buildType":"docker","builder":"GitHub Actions","source":"${{ github.repository }}"}'
          push-to-registry: true
          create-storage-record: true
          show-summary: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
