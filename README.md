name: Generate profile assets

on:
  schedule:
    # every 12 hours
    - cron: "0 */12 * * *"
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  # ---------------------------------------------------------------
  # Contribution snake -> pushed to the `output` branch as SVG
  # ---------------------------------------------------------------
  snake:
    runs-on: ubuntu-latest
    steps:
      - name: Generate snake animation
        uses: Platane/snk@v3
        id: snake-gif
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/snake.svg?palette=github-light
            dist/snake-dark.svg?palette=github-dark&color_snake=#255E63

      - name: Push to output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # ---------------------------------------------------------------
  # 3D contribution calendar -> committed to main under
  # profile-3d-contrib/
  # ---------------------------------------------------------------
  contrib-3d:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate 3D calendar
        uses: yoshi389111/github-profile-3d-contrib@0.7.1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          USERNAME: ${{ github.repository_owner }}

      - name: Commit generated SVGs
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add -A profile-3d-contrib
          git diff --quiet && git diff --staged --quiet || \
            git commit -m "chore: refresh 3D contribution calendar"
          git push
