# Book XI - Postulates

Postulates of Book XI are facts about front-ends in **our** estate. They could be otherwise elsewhere. Every proposition of Book XI rests on at least one of them; if one changes, re-examine the propositions that cite it.

---

**Post. XI.1 - Mixed Technology.** Front-ends are built with a mixture of technologies and this mixture will persist (Post. I.2). Book XI is therefore framework-neutral: every proposition must be satisfiable by any front-end that can issue HTTP requests and parse JSON, and no proposition may name a framework in its Statement.

**Post. XI.2 - Untrusted Clients.** Browsers and native applications run on devices the company does not control. Any code, secret or check placed in a front-end can be read, altered or bypassed by the person holding the device. It follows that a front-end can enforce nothing; it can only present, validate for convenience, and request.

**Post. XI.3 - Contracts Only.** Front-ends consume only Book II API contracts and Book IV shared schemas. A front-end never reaches a database, a message queue, an internal endpoint, or a service listener that is not behind the edge (Post. V.2). This is Post. I.3 applied to a consumer that is not a service.

**Post. XI.4 - Common Language.** TypeScript is the language every front-end technology in the estate can consume, whatever framework it uses. Shared front-end packages (generated clients, contract types, error maps) are therefore published as TypeScript and require nothing else. *(Open question: confirm that no front-end in the estate is unable to consume a TypeScript package; recorded in the Book XI README.)*
