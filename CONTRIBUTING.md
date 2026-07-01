# Přispívání k dokumentaci

Pokud chcete udělat dobrý skutek a přidat něco do tohoto repozitáře, nebo jen
něco menšího opravit, díky! [Vytvořte
issue](https://github.com/bakalari-api/bakalari-api-v3/issues), forkněte tohle
repo, přidejte do svého forku změny a pak už jen vytvořte PR odkazující na vaše
issue.

Vaše změny však musí být správně zformátované. Pokud Váš editor podporuje
[`EditorConfig`](https://editorconfig.org/) (Visual Studio Code, Neovim...), měl
by spoustu věcí ohledně formátování dělat sám od sebe. Každopádně ale
potřebujete [Prettier](https://prettier.io/); Pokud používáte Visual Studio
Code, je pro Vás [plugin dostupný na tržišti s
rozšířeními](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode).
Až dopíšete své změny, zmáčkněte v dokumentu `Ctrl+Shift+P` a vyberte možnost
`Format document`/`Formátovat dokument`. Pokud používáte něco jiného, tak to
vůbec nevadí. Ozbrojte se svým oblíbeným správcem balíčků pro JavaScript a
spusťte Prettier z shellu:

```bash
npx prettier@latest --write ./**/*.{md,json}
```

Zde jsem použil `npx`, protože je nejpopulárnější, ale použijte si, co chcete.

