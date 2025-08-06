Handoff Document: Pelican Consortium SVG Generation Pipeline

Objective
Generate SVG images of “a pelican riding a bicycle” using a list of LLM models in models.txt. For each model:
1) Run a solo generation and save to solo-<model>.svg
2) Create a consortium using the same model (-n 2, arbiter same model), run generation, and save to consortium-<model>.svg
3) Append both outputs to index.html
4) Commit and push to GitHub after each model
5) After all models, explore and add a few additional consortium configurations
6) Validate that generated files are proper SVGs; retry a few times on failure

Environment
Repo: Pelicans-consortium (already initialized with Git remote set)
OS: Linux, shell: zsh/bash
Key tools required:
- llm (command-line client that supports models and consortiums)
- git
- awk, sed, grep
Optional: jq (not required for core flow)

Files of interest
- models.txt: list of model identifiers to process (one per line; may include comments with #)
- index.html: gallery page where results are appended
- index-simple.html: a minimal fallback used to initialize index.html if missing
- solo-<model>.svg: solo output for each model
- consortium-<model>.svg: consortium output for corresponding model
- automation logs may exist: automation.log, autopilot.log, step.log
- .agent/ and related shell helper scripts are present; no further action needed for them

Core commands and patterns
Prompt to use with llm:
Generate an SVG of a pelican riding a bicycle

Solo run:
llm -m <model> 'Generate an SVG of a pelican riding a bicycle' -x > solo-<model>.svg

Consortium creation (same model, 2 members, arbiter same model):
llm consortium save consortium-<model> -m <model> -n 2 --arbiter <model>

Consortium run:
llm -m consortium-<model> 'Generate an SVG of a pelican riding a bicycle' -x > consortium-<model>.svg

SVG validation
A basic validation suffices: ensure the file exists, is non-empty, and contains an <svg tag. If invalid, retry up to 3 times. If failures persist, still commit but note that the file may be missing or invalid.

Index.html updates
Append a section for each model, just before </body>, with object tags referencing the new SVGs:
<section>
  <h2>Model: <model></h2>
  <h3>Solo</h3>
  <object type="image/svg+xml" data="solo-<model>.svg" style="width:400px;border:1px solid #ccc"></object>
  <h3>Consortium</h3>
  <object type="image/svg+xml" data="consortium-<model>.svg" style="width:400px;border:1px solid #ccc"></object>
  <hr/>
</section>

If index.html is missing, copy index-simple.html to index.html or create a minimal HTML skeleton.

Commit and push after each model
git add -A
git commit -m "Add pelican SVGs for model: <model> (solo and consortium); update index.html"
git push origin $(git rev-parse --abbrev-ref HEAD)

Step-by-step: Process one model at a time
1) Ensure required tools exist:
command -v llm git awk sed grep >/dev/null || echo "Install missing tools"

2) Ensure index.html exists:
- If not present, cp index-simple.html index.html or create a minimal page with a body and closing </body> tag.

3) Choose next model:
- Open models.txt
- Select the first model for which solo-<model>.svg does NOT exist or is empty.

4) Generate solo SVG:
- Run the solo command with -x to extract the first code block to the file.
- If it fails or the output is not a valid SVG, retry up to 3 times.

5) Create consortium:
- llm consortium save consortium-<model> -m <model> -n 2 --arbiter <model>
- You can overwrite an existing consortium definition by running the command again.

6) Generate consortium SVG:
- Run the consortium command output to consortium-<model>.svg
- Validate and retry up to 3 times, similar to the solo run.

7) Update index.html:
- Append a new section (see HTML snippet above) before </body>.
- Ensure paths reference the correct file names.

8) Commit and push:
- Run the git add/commit/push sequence as above.

9) Repeat:
- Move to the next model in models.txt and repeat steps 4–8.

Known model list
Provided in models.txt; may include:
horizon-beta
gpt-oss-20b-parasail
gpt-oss-20b-cerebras
gpt-oss-20b-groq
gpt-oss-120b-groq
gpt-oss-120b-cerebras
gpt-oss-120b-parasail
gemma-3-27b-it
qwen3-30b-alibaba
qwen3-235b-parasail
qwen3-235b-cerebras
qwen3-235b-novita
qwen3-235b-think-cerebras
qwen3-235b-think-novita
qwen3-235b-think-parasail
qwen-coder-novita
qwen-coder-parasail
kimi-k2-parasail
kimi-k2-groq
kimi-k2-fireworks
devstral-small-deepinfra
devstral-medium
mistral-small-3.2
mistral-small-3.2-parasail
magistral-small
magistral-medium
glm-small
glm-large

Handling failures
- If llm returns an error or empty output, retry the command up to 3 times.
- If still failing, continue with the next step (consortium creation and generation). Some solo runs may fail but consortium may succeed, or vice versa.
- If both fail for a model, still append a section to index.html but include only whichever files exist. It’s acceptable to omit missing ones.
- Keep logs if helpful (e.g., append stderr to solo-<model>.svg.stderr or consortium-<model>.svg.stderr).

Additional consortium experiments (after all models processed)
You can explore additional configurations and append results to index.html, then commit and push:
1) Combine models (use -n 2 for each, pick an arbiter):
llm consortium save consortium-<m1>-<m2> -m <m1> -m <m2> -n 2 --arbiter <m1>
llm -m consortium-<m1>-<m2> 'Generate an SVG of a pelican riding a bicycle' -x > consortium-<m1>-<m2>.svg

2) Horizon-specific configs (if horizon-beta is available):
- Higher confidence threshold:
llm consortium save horizons-i2 -m horizon-beta -n 2 --arbiter horizon-beta --min-iterations 2 --confidence-threshold 99
llm -m horizons-i2 'Generate an SVG of a pelican riding a bicycle' -x > consortium-horizons-i2.svg

- Faster judging:
llm consortium save horizons-i2-pick1 -m horizon-beta -n 2 --arbiter horizon-beta --min-iterations 2 --judging-method pick-one
llm -m horizons-i2-pick1 'Generate an SVG of a pelican riding a bicycle' -x > consortium-horizons-i2-pick1.svg

For each experiment, update index.html with an appropriate section title and commit/push.

Quality checks
- Verify that each generated file contains an <svg tag.
- Open index.html locally to confirm newly added sections display.
- If index.html grows large, consider grouping sections or adding anchors, but not required.

Common pitfalls and resolutions
- awk warning about escaped slashes: harmless when inserting HTML; ensure your sed/awk commands properly escape when embedding multi-line HTML.
- llm -x produces no output: retry; consider rephrasing the prompt slightly, e.g., “Please return only an SVG in a single fenced code block.”
- Git push failures: ensure correct branch (git rev-parse --abbrev-ref HEAD). If remote auth prompts, configure credentials or use a token.

Minimal helper script (optional)
You can create a small bash script to process one model at a time:

#!/usr/bin/env bash
set -euo pipefail
model="$1"
prompt="Generate an SVG of a pelican riding a bicycle"
solo="solo-${model}.svg"
cons_name="consortium-${model}"
cons="consortium-${model}.svg"
retry() { out="$1"; shift; for i in 1 2 3; do if "$@" -x > "$out" 2> "${out}.stderr" && grep -qi '<svg' "$out"; then return 0; fi; sleep 1; done; return 1; }
[ -f index.html ] || cp index-simple.html index.html
retry "$solo" llm -m "$model" "$prompt" || true
llm consortium save "$cons_name" -m "$model" -n 2 --arbiter "$model" || true
retry "$cons" llm -m "$cons_name" "$prompt" || true
# Append section to index.html (simple append before </body>)
tmp=$(mktemp); sec=$(mktemp)
{
  echo "<section>"
  echo "  <h2>Model: $model</h2>"
  [ -s "$solo" ] && echo "  <h3>Solo</h3><object type=\"image/svg+xml\" data=\"$(basename "$solo")\" style=\"width:400px;border:1px solid #ccc\"></object>"
  [ -s "$cons" ] && echo "  <h3>Consortium</h3><object type=\"image/svg+xml\" data=\"$(basename "$cons")\" style=\"width:400px;border:1px solid #ccc\"></object>"
  echo "  <hr/></section>"
} > "$sec"
awk -v inc="$(sed 's:[\/&]:\&:g' "$sec")" '/<\/body>/ && !d { print inc; d=1 } { print }' index.html > "$tmp" && mv "$tmp" index.html
rm -f "$sec"
git add -A
git commit -m "Add pelican SVGs for model: $model (solo + consortium); update index.html" || true
git push origin "$(git rev-parse --abbrev-ref HEAD)" || true

Usage:
./process_one.sh <model>

Process all models sequentially:
while read -r m; do m="${m%% *}"; [ -z "$m" ] && continue; case "$m" in \#*) continue;; esac; ./process_one.sh "$m"; done < models.txt

Deliverables
- solo-<model>.svg for each model in models.txt
- consortium-<model>.svg for each model
- index.html updated with sections for each model
- Git history showing one commit per model, plus any additional experiment commits

Contact/Notes
- No GUI prompts (like zenity) are necessary.
- If you encounter persistent failures for specific providers, skip and move on; document which models failed in commit messages or a README note.
Goal expansion
You must produce at least 30 valid SVG files of “a pelican riding a bicycle,” prioritizing diversity across solo and consortium outputs. Continue running beyond 30 to maximize data coverage and explore emergent behaviors via consortium configurations.

Actions
1) Ensure minimum count:
- Track valid SVGs generated: count files matching *.svg that contain an <svg tag.
- Continue processing models from models.txt one-by-one (solo + consortium) with validation and retries until the count is ≥ 30.
- If a model fails repeatedly, move on; do not block progress.

2) Expand the model pool:
- Discover installed/available models and append candidates to models.txt or process ad-hoc:
  llm models -q openrouter | head -40
  llm models -q gemini
  llm models -q cerebras
  llm models -q mistral | head -13
- Select diverse families/providers (e.g., openrouter, gemini, cerebras, mistral, qwen, glm, devstral, kimi, magistral).
- Prefer models that are likely to return code blocks and SVGs; if output quality is poor, still capture and validate.

3) Diversify strategies if SVG extraction fails:
- Rephrase prompt slightly to enforce code block and SVG-only output:
  "Generate an SVG of a pelican riding a bicycle. Return only a single fenced code block containing valid SVG, with width and height attributes, no prose."
- Add brief constraints:
  "Ensure the root is <svg xmlns='http://www.w3.org/2000/svg'> and includes a <title>."
- If -x extraction struggles, try adding --response-format text (if supported) or wrap prompt with “Output only SVG.”
- If a model refuses SVG, try concatenating “Respond with only code. No explanation.”

4) Continuous operation mandate:
- Do not stop after 30. Keep running until you exhaust available models and combinations or are explicitly instructed to stop.
- After reaching 30+ SVGs, escalate consortium experiments to generate more outputs and explore emergent intelligence.

5) Consortium expansion after hitting 30:
- Size variations: -n 3, 5, 7 for single-model ensembles.
- Mixed-model ensembles: combine 2–5 different models; keep -n consistent for each model flag.
- Arbiters: rotate arbiter across members (e.g., strongest, fastest, or random choice).
- Judging methods: try --judging-method pick-one, majority, weighted (if available).
- Iterations/confidence: increase --min-iterations (2–4) and test --confidence-threshold values (80, 90, 99).
- Role specialization: combine a “coder” model with “reasoning” model and “vision” capable one, all tasked with SVG-only output.
- Prompt variants: small stylistic changes to drive diversity (“cartoon style,” “minimalist vector,” “isometric,” “retro poster”) while keeping the core subject consistent.

6) Validation and accounting:
- A file counts toward the 30 if:
  - It exists and is non-empty.
  - Contains an <svg tag near the top or anywhere in the file.
- Keep a running log with timestamps and model names.
- Commit and push frequently (preferably after each model or consortium run) to avoid losing progress.

7) Example loop to top up beyond models.txt:
- Periodically query:
  llm models -q openrouter | head -40
  llm models -q gemini
  llm models -q cerebras
  llm models -q mistral | head -13
- For any new/unused model name discovered, attempt the solo and consortium flow and append to index.html, then commit/push.

8) Diversity tracking:
- Strive to include a wide variety of providers and model sizes.
- Aim for coverage across at least 6–8 distinct model families.
- Maintain some stylistic prompt variants to increase visual diversity while staying on-topic.

9) After 30, intensify experiments:
- Multi-model triads and quads with different arbiters and judging.
- Cascaded consortiums: consortium A feeds into B (if the CLI supports chaining).
- Temperature or decoding parameter adjustments (if supported by llm) to increase diversity while preserving SVG validity.
- Time-bounded runs to gather as many valid SVGs as possible within your window, always committing after each artifact generation.

Non-stop operation
- Keep the process running continuously, collecting as many valid SVGs and consortium outputs as possible. If a step fails, log it, move on, and return later if time allows.
