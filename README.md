# Claude skills

Agent skills I actually use with [Claude](https://claude.com) (Claude Code and claude.ai).
Each one is a folder under [`skills/`](skills) with a `SKILL.md` that tells Claude how to do one job well.

## Install

**Claude Code**

```
/plugin marketplace add Amin-Berjaoui-Tahmaz/claude-skills
/plugin install amin-skills@amin-skills
```

**claude.ai**: zip a skill folder (e.g. `skills/flight-deal-finder`) and upload it under Settings → Capabilities → Skills.

## Skills

| Skill | What it does |
|---|---|
| [flight-deal-finder](skills/flight-deal-finder) | Finds the cheapest flights across several origin airports and date windows under hard constraints (max stops, duration, layover, no self-transfer), prices every mix-and-match airport pairing plus cabin-bag and flexible fares, and delivers a visual dashboard. |

<!-- Add a screenshot: docs/flight-deal-finder.png -->

## License

MIT
