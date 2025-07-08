# RAGPack: Distributed Dataset Packaging and Discovery for RAG Systems

## Summary

**RAGPack** is a CLI and package format standard for building portable, shareable, and locally-queryable knowledge bundles for Retrieval-Augmented Generation (RAG) systems. It eliminates redundant embedding, web scraping, and chaotic dataset versioning by allowing users to share `.ragpack` files like Python packages.

## Key Features

- 📦 `.ragpack` format: documents + embeddings + metadata
- 🛠️ CLI tool: create, chunk, embed, query, pack, sign, verify, version
- 🔐 GPG-based signing and trust
- 🔢 Semantic versioning support
- 🌐 Optional FastAPI registry backend for hosting/discovery

---

## Directory Structure

```text
my-ragpack/
├── rag.json         # Metadata (name, license, embedding params, SPDX ID, version)
├── docs/            # Source documents
├── chunks.jsonl     # Chunked text
├── embeddings.npy   # Embeddings
├── signature.asc    # GPG signature (optional)
```

---

## CLI Scaffold (`ragpack.py`)

```python
import typer
from pathlib import Path
import json
import os
import shutil
import semver
import subprocess

app = typer.Typer()

@app.command()
def init(name: str):
    os.makedirs(name + "/docs", exist_ok=True)
    meta = {
        "name": name,
        "license": "CC-BY-4.0",
        "version": "0.1.0",
        "embedding": {"model": "all-MiniLM-L6-v2", "chunk_size": 512}
    }
    Path(name + "/rag.json").write_text(json.dumps(meta, indent=2))

@app.command()
def chunk(path: str):
    import jsonlines
    chunks = []
    ragpath = Path(path)
    for file in (ragpath / "docs").glob("*.txt"):
        text = file.read_text()
        for i in range(0, len(text), 512):
            chunks.append({"text": text[i:i+512], "source": file.name})
    with jsonlines.open(ragpath / "chunks.jsonl", mode='w') as writer:
        writer.write_all(chunks)

@app.command()
def embed(path: str):
    from sentence_transformers import SentenceTransformer
    import numpy as np
    import jsonlines
    ragpath = Path(path)
    model = SentenceTransformer("all-MiniLM-L6-v2")
    with jsonlines.open(ragpath / "chunks.jsonl") as reader:
        chunks = [r["text"] for r in reader]
    embeddings = model.encode(chunks)
    np.save(ragpath / "embeddings.npy", embeddings)

@app.command()
def query(path: str, prompt: str):
    import numpy as np
    from sentence_transformers import SentenceTransformer
    from sklearn.metrics.pairwise import cosine_similarity
    import jsonlines
    ragpath = Path(path)
    model = SentenceTransformer("all-MiniLM-L6-v2")
    qvec = model.encode([prompt])
    embs = np.load(ragpath / "embeddings.npy")
    with jsonlines.open(ragpath / "chunks.jsonl") as reader:
        chunks = list(reader)
    sims = cosine_similarity([qvec[0]], embs)[0]
    top = sorted(zip(sims, chunks), reverse=True)[:3]
    for score, chunk in top:
        print(f"[{score:.2f}] {chunk['text'][:100]}")

@app.command()
def pack(path: str):
    shutil.make_archive(path, 'zip', path)
    shutil.move(f"{path}.zip", f"{path}.ragpack")

@app.command()
def install(pkg: str, dest: str = "./packs"):
    os.makedirs(dest, exist_ok=True)
    shutil.unpack_archive(pkg, extract_dir=Path(dest) / Path(pkg).stem)

@app.command()
def push(path: str, url: str):
    import requests
    with open(f"{path}.ragpack", "rb") as f:
        r = requests.post(url + "/upload", files={"file": f})
        print(r.text)

@app.command()
def pull(name: str, url: str, dest: str = "./packs"):
    import requests
    r = requests.get(f"{url}/packs/{name}.ragpack")
    path = Path(dest) / f"{name}.ragpack"
    path.write_bytes(r.content)
    shutil.unpack_archive(path, extract_dir=path.parent / name)

@app.command()
def sign(path: str):
    subprocess.run(["gpg", "--armor", "--output", f"{path}/signature.asc", "--detach-sign", f"{path}/rag.json"])

@app.command()
def verify(path: str):
    subprocess.run(["gpg", "--verify", f"{path}/signature.asc", f"{path}/rag.json"])

@app.command()
def diff(p1: str, p2: str):
    j1 = json.loads(Path(p1, "rag.json").read_text())
    j2 = json.loads(Path(p2, "rag.json").read_text())
    for k in j1:
        if j1[k] != j2.get(k):
            print(f"{k}: {j1[k]} -> {j2.get(k)}")

@app.command()
def version(path: str, bump: str = "patch"):
    p = Path(path) / "rag.json"
    meta = json.loads(p.read_text())
    current = meta.get("version", "0.1.0")
    if bump == "major":
        new_ver = semver.VersionInfo.parse(current).bump_major()
    elif bump == "minor":
        new_ver = semver.VersionInfo.parse(current).bump_minor()
    else:
        new_ver = semver.VersionInfo.parse(current).bump_patch()
    meta["version"] = str(new_ver)
    p.write_text(json.dumps(meta, indent=2))
    typer.echo(f"Updated version: {new_ver}")

if __name__ == "__main__":
    app()
```

---

## FastAPI Registry Server (`registry.py`)

```python
from fastapi import FastAPI, UploadFile
from fastapi.responses import FileResponse
import os

app = FastAPI()

@app.post("/upload")
def upload(file: UploadFile):
    path = f"packs/{file.filename}"
    os.makedirs("packs", exist_ok=True)
    with open(path, "wb") as f:
        f.write(file.file.read())
    return {"status": "uploaded", "file": file.filename}

@app.get("/packs/{name}")
def download(name: str):
    return FileResponse(f"packs/{name}")
```

---

## Usage Example

```bash
# Create and use your first pack
python ragpack.py init my-pack
python ragpack.py chunk my-pack
python ragpack.py embed my-pack
python ragpack.py query my-pack --prompt "What is RAG?"
python ragpack.py pack my-pack
python ragpack.py sign my-pack
python ragpack.py verify my-pack
python ragpack.py push my-pack http://localhost:8000
python ragpack.py pull my-pack http://localhost:8000
```

---

## Roadmap

- ✅ MVP CLI (init, embed, query, pack, install, push, pull)
- ✅ GPG-based signing and verification
- ✅ Versioning and semver tracking
- 🔜 Web UI for registry
- 🔜 Ollama/LLM local integration

---

## License

MIT License

---

## Related Tools

- [sentence-transformers](https://www.sbert.net/)
- [FAISS](https://github.com/facebookresearch/faiss)
- [SPDX](https://spdx.dev/)


