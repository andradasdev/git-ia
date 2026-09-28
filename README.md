# Git, GitLab e IA aplicada à Engenharia de Software

![Git](https://img.shields.io/badge/Git-GitLab_Self--Managed-F05032?logo=git&logoColor=white)
![Claude Code](https://img.shields.io/badge/IA-Claude_Code-D97757)
![Publicação](https://img.shields.io/badge/GitHub_Pages-publicado-050F41)
![Licença](https://img.shields.io/badge/conteúdo-CC_BY--NC--SA_4.0-lightgrey)

![Git, GitLab e IA aplicada à Engenharia de Software](assets/card.png)

Material público de uma capacitação de **1 hora** que preparei durante o meu estágio em Análise e Desenvolvimento de Sistemas na **Marinha do Brasil**. Ela parte de dois problemas reais:

1. **integrar um sistema que já tinha histórico Git a um GitLab Self-Managed** em que a `main` já existe e está protegida;
2. **documentar sistemas legados com um agente de IA** (Claude Code), usando um prompt que evoluiu a partir das próprias falhas.

### 📖 **[Acessar o material](https://andradasdev.github.io/git-ia/)**

| | |
|---|---|
| 📘 Apostila para iniciantes, com exercícios | [`docs/apostila.pdf`](docs/apostila.pdf) |
| 🖥️ Slides da apresentação | [`docs/apresentacao.pdf`](docs/apresentacao.pdf) |

---

## 🧭 Este repositório é só de publicação

Aqui ficam apenas os **artefatos prontos**. Eles não são editados neste repositório: chegam por automação, compilados a partir das fontes LaTeX.

> ### 👉 Para ver o código-fonte, ir ao repositório de código:
> ## **[github.com/ronidomingues/git-ia-docs](https://github.com/ronidomingues/git-ia-docs)**
>
> Lá estão as fontes `.tex` da apostila, dos slides e do roteiro de falas, o tema LaTeX, o prompt final de documentação e o histórico de desenvolvimento. **Correções e contribuições devem ser abertas lá.** Qualquer alteração feita diretamente aqui é sobrescrita na próxima publicação.

### Como o material chega até aqui

```
  ronidomingues/git-ia-docs                       andradasdev/git-ia
  ─────────────────────────                       ──────────────────
   fontes .tex, tema, prompt
            │
            │  push na main
            v
   ┌──────────────────────┐
   │ workflow: build.yml  │
   │  1. compila LaTeX    │
   │  2. verifica os PDFs │
   │  3. commita lá       │
   │  4. envia para cá ───┼──────────────>  apostila.pdf
   └──────────────────────┘                 apresentacao.pdf
                                            card.png
                                                  │
                                                  │ dispara
                                                  v
                                        ┌──────────────────────┐
                                        │ workflow: pages.yml  │
                                        │  publica o site      │
                                        └──────────┬───────────┘
                                                   v
                                    andradasdev.github.io/git-ia
```

A autenticação entre os dois repositórios está documentada em **[`documentacao/autenticacao-github-actions.md`](documentacao/autenticacao-github-actions.md)**.

---

## 🎯 O que a capacitação cobre

Ao final, o participante é capaz de:

1. diferenciar Git, GitLab, GitHub e o GitLab CLI (`glab`);
2. explicar commit, branch, merge e Merge Request;
3. entender e resolver um conflito;
4. unir dois históricos Git independentes com `--allow-unrelated-histories`, escolhendo com consciência entre `ours` e `theirs`;
5. usar um agente de IA como ferramenta de engenharia, com descoberta somente leitura, a regra de nunca inventar, o protocolo de parar e reportar, e validação humana.

A apostila mede como o prompt de documentação evoluiu em **5 versões**, a partir de **717 prompts reais**, e mostra como cada falha observada virou uma regra.

### Os 60 minutos

| # | Bloco | Início | Duração |
|---|---|---|---|
| 0 | Abertura, agenda e objetivos | 00:00 | 3 min |
| 1 | Fundamentos: Git, GitLab, Self-Managed, branch, Merge Request, conflito, `glab` | 00:03 | 19 min |
| 2 | Problema 1: dois históricos sem relação, os oito passos, `ours` e `theirs` | 00:22 | 15 min |
| 3 | Problema 2: IA como ferramenta, evolução dos prompts, falhas e regras, travas de segurança | 00:37 | 18 min |
| 4 | Encerramento e perguntas | 00:55 | 5 min |
| | **Total** | | **60 min** |

A apresentação é só expositiva. Os exercícios práticos estão na apostila, e um deles reproduz o Problema 1 inteiro no seu computador, sem servidor.

---

## 📂 Estrutura

```
git-ia/
├── docs/
│   ├── apostila.pdf          apostila (gerada, não editar aqui)
│   └── apresentacao.pdf      slides (gerados, não editar aqui)
├── assets/
│   └── card.png              imagem de prévia do site (gerada)
├── documentacao/
│   └── autenticacao-github-actions.md
├── index.html                página publicada no GitHub Pages
├── LICENSE                   MIT, para o código
├── LICENSE-CONTENT           CC BY-NC-SA 4.0, para o conteúdo didático
└── README.md
```

---

## ⚖️ Licença

- **Conteúdo didático** (apostila, slides, textos, diagramas, exercícios): [CC BY-NC-SA 4.0](LICENSE-CONTENT)
- **Código** (workflows, `index.html`): [MIT](LICENSE)

Você pode usar, adaptar e reaplicar este material para fins não comerciais, desde que dê o crédito e mantenha a mesma licença. Imagens e fontes de terceiros têm licença própria, listada no repositório de código.

Material pessoal, com publicação autorizada. Não é publicação oficial da Marinha do Brasil.

## 👨‍🏫 Autor

**Ronivaldo Domingues de Andrade**
LinkedIn: [ronidomingues](https://www.linkedin.com/in/ronidomingues/) ·
GitHub: [@ronidomingues](https://github.com/ronidomingues)
📍 Rio de Janeiro, RJ

### ⭐ Se este material foi útil, considere dar uma estrela no repositório!
