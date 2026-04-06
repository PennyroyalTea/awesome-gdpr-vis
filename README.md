# Awesome GDPR vis skills

A collection of Claude skills for visualising your personal data from GDPR/SAR exports.

Each app has a `skill.md` -- a prompt-based guide that tells Claude how to build interactive dashboards from your data. No build tools needed, everything runs as plain HTML with CDN dependencies.

<p align="center">
  <img src="lime/screenshots/animated.gif" alt="Example: Lime rides data vis" />
  <br>
  <sub>Example: Lime rides data vis</sub>
</p>

## Apps

| App | What you get | Data request |
|-----|-------------|--------------|
| [Lime](lime/) | Ride maps (6 modes), dashboard, speed trends, surveillance analysis | [privacy.li.me](https://privacy.li.me/) |

## How it works

1. Request your data from the app (GDPR Subject Access Request)
2. Pick the matching skill from this repo
3. Start a Claude session, share the `skill.md` and your data files
4. Ask Claude to build the dashboards

## Contributing

To add a new app, open a PR with a directory containing:
- `skill.md` -- detailed instructions for Claude on how to process and visualise the data
- `README.md` -- what the app is, what you get, how to request your data
- `screenshots/` -- example output
