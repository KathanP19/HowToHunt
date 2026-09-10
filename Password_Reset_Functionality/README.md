# Password Reset Functionality — Index

Password reset is the classic Account Takeover entry point: it authenticates via email-borne secrets instead of sessions. This folder splits the flaw classes into focused playbooks so each can be hunted and reported separately. Severity ceiling of this folder: P1 (reset-token theft/AATO).

## Hunting Checklist (folder routing)

1. Start with the five highest-hit-rate flaws end to end: `Top_5_Password_Reset_Bugs.md`.
2. Run the full flaw-class sweep: `Password_Reset_Flaws_by_Sm4rty.md`.
3. Abuse the email parameter itself: `Account_Takeover_By_Password_Reset_Functionality.md`.
4. Audit every leak channel for the token: `Password_Reset_Token_Leakage.md`.
5. Test token/session lifecycle (expiry, single-use, revocation): `../Broken_Auth_And_Session_Management/Session_based_bugs.md` §§6–8.

## Test Cases & Examples

Per-file — payloads are kept exactly once (cross-referenced, never duplicated):
- Requests/payloads → `Top_5_Password_Reset_Bugs.md`, `Account_Takeover_By_Password_Reset_Functionality.md`
- Step-by-step methods → each file's Hunting Checklist.

## Tools & Resources

- Mindmap (N008x):

![Password Reset Mindmap](https://pbs.twimg.com/media/EhN29bpU8AMvLxx?format=jpg&name=medium)

- Source: [Twitter](https://twitter.com/N008x/status/1302515523557548032/photo/1)
- Source: [Blog — 10 Password Reset Flaws](https://anugrahsr.github.io/posts/10-Password-reset-flaws/)

## Report Tips

- One flaw class per report; chain the others as "additional evidence" bullets.
- Always demonstrate against a second account you control as the "victim".

## Credits

- [KathanP19](https://twitter.com/KathanP19)
