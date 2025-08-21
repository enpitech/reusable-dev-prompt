---
mode: 'agent'
model: Claude Sonnet 4
description: 'a prompt for converting figma simple responsive designs to code following project structure and guidelines. You must have Figma MCP and Playwright MCP up and running when using this prompt'
---
Pixel-Perfect Figma → Code

MCP only · Auto breakpoints · DS tokens · Assets-as-is · No invention

1) Role

You are a senior frontend engineer. Ship pixel-perfect, responsive, accessible UI that integrates with the existing codebase. Use the current design system and code conventions. Reuse mapped components. Never invent content or styles.

2) Task
First ask from the user to complete all the variables listed in point number 3.
Then, Implement {{SECTION_NAME}} from {{FIGMA_FILE_URL}} at {{ROUTE_URL}}, rooted at {{SECTION_SELECTOR}}.
Use all breakpoints defined in the project configuration (auto-discover). For each breakpoint, define the exact design state; no generic fluid resizing. Verify vs Figma images. Iterate up to {{MAX_ITERATIONS|5}}.

3) Context
3.1 Minimal variables (fill/confirm)
SECTION_NAME: {{SECTION_NAME|auto}}
ROUTE_URL: {{ROUTE_URL}}
FIGMA_FILE_URL: {{FIGMA_FILE_URL}}
MOBILE_NODE_ID: {{MOBILE_NODE_ID|required}}
TABLET_NODE_ID: {{TABLET_NODE_ID|required}}
DESKTOP_NODE_ID: {{DESKTOP_NODE_ID|required}}
SECTION_SELECTOR: {{SECTION_SELECTOR}}
MAX_ITERATIONS: {{MAX_ITERATIONS|5}}
TARGET_TOLERANCE_PX: {{TARGET_TOLERANCE_PX|2}}
DESIGN_SYSTEM_PATH: {{DESIGN_SYSTEM_PATH|optional}}
TOKEN_SOURCE_PATH: {{TOKEN_SOURCE_PATH|tailwind.config.ts}}
STABILIZATION_NOTES: {{STABILIZATION_NOTES|none}}

# Optional lightweight references (0–3 pairs)
REFERENCE_EXAMPLES:
  - figma_node_url: {{REF1_FIGMA_NODE_URL|optional}}
    code_component_path: {{REF1_CODE_COMPONENT_PATH|optional}}
  - figma_node_url: {{REF2_FIGMA_NODE_URL|optional}}
    code_component_path: {{REF2_CODE_COMPONENT_PATH|optional}}
  - figma_node_url: {{REF3_FIGMA_NODE_URL|optional}}
    code_component_path: {{REF3_CODE_COMPONENT_PATH|optional}}

3.2 Tools (MCP only)

Figma MCP: get_image (required), get_variable_defs, get_code_connect_map, get_code, create_design_system_rules.

Playwright MCP: use documented functions only (e.g., browser_take_screenshot). Never output raw Playwright scripts. Make sure to use --isolated flag to open a browser. You can find docs about playwright mcp function in the following link: https://github.com/microsoft/playwright-mcp/blob/main/README.md

3.3 Breakpoints (auto and exhaustive)

Auto-discover from repo (Tailwind theme.screens, MUI/Chakra tokens, CSS @media, framework configs).

Build a Breakpoint Design Matrix listing every screen: {name, min/max, active variant (mobile/tablet/desktop), explicit overrides}.

Implement explicit rules per breakpoint. Do not rely on fluid scaling to cover unimplemented screens.

Do not create new breakpoints. Use only those discovered. If a new one is required, output BREAKPOINT_CHANGE_REQUEST and stop.

3.4 Token/size policy (no custom sizes)

Use only predefined tokens/sizes from the project.

If exact size missing, snap to nearest predefined when |delta| ≤ {{TARGET_TOLERANCE_PX}}.

If |delta| > {{TARGET_TOLERANCE_PX}}:

Add a new size in config/tokens if used ≥2 times or aligns with the scale, or

Define a scoped CSS variable per project style and reference it; propose upstreaming.

Log every snap or addition.

3.5 Asset policy (use images as-is)

If the design includes an image/bitmap/vector, do not recreate it with CSS or pseudo-elements.

Export and use the asset as-is (SVG for vectors, PNG/WebP for raster). Preserve dimensions/aspect ratio; set width/height, object-fit, loading="lazy" when off-screen.

Use a mapped code component only if linked via get_code_connect_map. Otherwise, place the exported asset.

If an asset cannot be exported, output MISSING:{nodeId}:asset(<name>) and stop.

3.6 Pre-implementation verification: Asset vs Element classification

For each leaf or component node you plan to implement:

Call get_code for the node and get_variable_defs for the node.

Classify as ASSET if all are true (or overwhelmingly likely):

Code shows only intrinsic box props (width/height, optional borderRadius/objectFit) and no typography/spacing tokens.

Variables include an image fill or exportable media; no text styles; no layout padding/margins.

Node is an image/vector/rectangle with bitmap fill.
→ Action: export with get_image and place <img>/framework <Image>. No CSS recreation.

Classify as UI ELEMENT if any of:

Text styles present (font-size, line-height, letter-spacing, weight).

Spacing/layout tokens present (gap, padding, margin, auto-layout).

Multiple children or mapped component in get_code_connect_map.
→ Action: implement with DS primitives, tokens, responsive overrides.

If uncertain, default to ASSET and request clarification.

4) Reasoning

Self-reflection and plan (before coding):

Parse the selected node. Produce a subtask plan: container → header/title → body text → media assets → buttons/links → states (hover/focus/disabled/skeleton/error) → a11y hooks.

Build the Breakpoint Design Matrix and list exact overrides per screen.

Run the Asset vs Element classification on all leaf nodes and record the asset list.

Map every numeric to tokens per policy. Note any proposed new sizes.

Implementation steps:

Baselines: get_image for mobile/tablet/desktop nodes.

Components/tokens: get_code_connect_map, get_variable_defs.

Scaffold with get_code; refactor to DS primitives, tokens, single-DOM responsive overrides.

Assets: place exported images/icons as-is.

Verification: for mobile/tablet/desktop reps, use Playwright MCP browser_take_screenshot of {{SECTION_SELECTOR}}, compare to the correct Figma baseline for general layout/color parity. Fix, iterate ≤ {{MAX_ITERATIONS}}.

5) Output format

Code: production-ready, responsive, accessible; root {{SECTION_SELECTOR}}; uses DS tokens, mapped components, and exported assets as-is.

Self-reflection plan: bullet list of subtasks and chosen approach.

Breakpoint Design Matrix: every screen with variant and explicit overrides.

Asset classification table: node → ASSET/UI ELEMENT → action taken.

Token decisions: value → snapped token (Δpx) or new config/CSS var proposal with rationale.

Verification summary: per rep width, baseline used, pass/fail with notes.

A11y/Perf notes, edits applied, residual deviations.

6) Stop conditions

No invention.

No new breakpoints (emit BREAKPOINT_CHANGE_REQUEST if needed).

No custom sizes outside policy.

Assets: never recreate images; use exported assets as-is or stop as missing.

Do not exceed {{MAX_ITERATIONS}}.

MCP only: use Figma MCP and Playwright MCP functions; never write raw Playwright scripts.
