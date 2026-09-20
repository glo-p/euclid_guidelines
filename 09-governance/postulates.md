# Book X - Postulates

Postulates of Book X are facts about how decisions are made in **our** organisation. They could be otherwise in another company. If one changes, re-examine every proposition that cites it.

---

**Post. X.1 - Three Principals Accept.** The principal engineer group (Def. X.7) consists of three people, and only they accept or reject a change to the guidelines (Cor. I.4.2). Acceptance requires two of the three. No other role, committee or manager can accept a change, and no principal can accept alone.

**Post. X.2 - Guidelines Live in Git.** The guidelines are one git repository. They change only by a merged pull request; a guideline that is not in the default branch is not in force. The repository history is the complete record of how the guidelines have changed.

**Post. X.3 - Every Service Has a Repository.** Every service (Def. I.1) has exactly one git repository, owned by its team (Def. I.17), with a CI pipeline that runs on every pull request and on every merge to the default branch. There is no service without a repository and no repository serving two services.
