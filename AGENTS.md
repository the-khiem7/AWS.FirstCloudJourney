# Skillset

## Encoding Hygiene
- All markdown stays UTF-8 (no BOM). If Vietnamese text shows mojibake (e.g., `lÃ `), run a one-time repair with `python -m pip install ftfy` and a small script using `ftfy.fix_encoding` on the affected files, then save back as UTF-8.
- Do not keep alternate encodings in the repo. The ftfy dependency is only for recovery; it is not required for normal builds.

## OCR Skill (local images)
- Install Tesseract (e.g., `choco install tesseract` or `winget install tesseract`) and ensure `tesseract.exe` is on PATH.
- Install Python deps: `pip install pytesseract pillow`.
- Set Tesseract data path if needed: `$env:TESSDATA_PREFIX="C:\Program Files\Tesseract-OCR\tessdata"`.
- OCR a local image (VI/EN):  
  ```powershell
  @'
  from PIL import Image
  import pytesseract, sys
  img_path = sys.argv[1]
  text = pytesseract.image_to_string(Image.open(img_path), lang="vie")  # use "eng" if English only
  print(text)
  '@ | python - "D:\path\to\image.png"
  ```

# Repository Guidelines

## Content Handling
- Use `content/` for markdown; avoid touching generated `public/`.
- Keep new sections paired when bilingual is required (`_index.md` and `_index.vi.md`), mirroring front matter keys and preserving structure.

## Build
- Run `npm run build` to verify Hugo output before shipping changes.