# jj-vcs skills

**Community-oriented skills for [Jujutsu (jj)](https://github.com/jj-vcs/jj).**

This snapshot is the **jj 0.40.0** baseline skill pack.

| | |
|---|---|
| **Skill pack** | [`using-jj/`](./using-jj/) |
| **Skills file** | [`using-jj/SKILL.md`](./using-jj/SKILL.md) |

Official docs: [v0.40.0](https://www.jj-vcs.dev/v0.40.0/)

## Install

Point skill discovery at a skills root that contains `using-jj/` (this tree’s
`using-jj/` folder), or symlink that folder into an existing skills root.

```bash
ln -s "$(pwd)/using-jj" /path/to/.agents/skills/using-jj
```

What matters: the tool sees a directory named `using-jj` that contains
**`SKILL.md`**.

## Version policy

Recipes in this snapshot target **jj 0.40.0**. Check the installed CLI with
`jj --version`. For newer CLI lines, use the matching tag or the default branch
on GitHub.

## License

[MIT](https://github.com/techsaint/jj-vcs-skills/blob/main/LICENSE) © [techsaint](https://github.com/techsaint)
