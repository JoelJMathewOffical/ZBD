# zero-bug-developement

A reusable skill for building software with defect prevention at its core. It applies NASA-style practices (requirements traceability, independent review, strict coding rules, static analysis gates, and risk-scaled rigor) to everyday development work.

The goal is zero known defects at release, plus removal of the bug classes that cause real damage. Literal zero bugs is not a realistic target for any software, so the skill focuses on layered prevention.

## What it covers

- **Risk tiers** (Critical, High, Standard, Low) that set how much process a change needs
- **Numbered requirements, invariants, and failure modes** written before any code
- **Bounded-code rules**: no unbounded recursion or loops, timeouts on external calls, checked return values, short functions
- **Test-first development** with boundary, invariant, and failure-mode tests, plus a regression test for every fix
- **Automated gates**: strict compiler and linter, security scanning, and no skipped tests in the merge branch
- **Independent review** with a checklist and a standard finding format
- **Release criteria** and a post-release defect loop
- **Anti-patterns** to flag, such as floats for money, client-only auth checks, and retries on non-idempotent writes

## Install

### Claude skill

Upload the `zero-bug-developement.zip` file, or copy the `SKILL.md` file into your skills folder. Claude will load it automatically when you ask for reliable, bug-free, or production-ready code, or when you request a review before merge.

### Manual use

Paste the contents of `SKILL.md` into a project's instructions or a system prompt, then ask for work as usual.

## Repository layout

```
ZBD/
├── SKILL.md     # The skill itself (the instructions Claude follows)
└── README.md    # This file
```

## Using it

Some example requests that trigger the skill:

- "Harden this payment flow for production."
- "Review this before I merge."
- "Make this reliable. It handles user balances."
- "Write this feature with a zero-bug approach."

For small teams, the minimum that removes most damage is: strict compiler and linter in CI, a requirement-to-test table, a written checklist before merging Critical or High code, and a regression test for every bug.

## Basis

The practices are drawn from NASA's Software Engineering Requirements (NPR 7150.2), JPL's "Power of Ten" coding rules for safety-critical C (Gerard Holzmann, 2006), and common industry practice for independent verification and static analysis.

## Contributing

Open an issue or pull request with the change and the reason for it. Changes to the rules should name the defect class they prevent.

## License

Add a license before making the repository public. If you want, I can draft an MIT license file for you to add.
