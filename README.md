# Colorful B360M BIOS Update Plugin

A Claude Code skill for updating BIOS on Colorful (七彩虹) Battle Axe B360M series motherboards via Intel FPT DOS method.

## Problem Solved

Colorful B360M-D and similar boards ship with a 2018 BIOS that lacks microcode for Intel 9th-gen CPUs (i7-9700F, i9-9900K, etc.). The standard Windows flash tool fails with **Error 3** due to Intel Flash Descriptor security. This skill guides users through the correct DOS-based FPT method.

## Applies To

- 七彩虹 战斧 C.B360M-D 魔音版 V20
- Colorful Battle Axe C.B360M-F / C.B360M-HD series
- Any Colorful Intel 300-series board with similar BIOS packaging

## What This Skill Does

- Diagnoses the root cause of frequent restarts (BugcheckCode=0 = hardware power loss)
- Explains why Windows FPT fails (Flash Descriptor security)
- Automates FreeDOS USB creation and file preparation
- Provides complete step-by-step BIOS flash instructions

## Installation

```bash
# Clone to your Claude plugins directory
git clone https://github.com/yangdonghua321/colorful-bios-update \
  ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/colorful-bios-update
```

## Usage

Once installed, Claude will automatically invoke this skill when you mention:
- "七彩虹 BIOS 更新"
- "Colorful B360M BIOS flash"
- "Intel FPT Error 3"
- Upgrading CPU on a Colorful B360 board

## Structure

```
colorful-bios-update/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── skills/
    └── colorful-b360m-bios-update/
        └── SKILL.md
```

## License

MIT
