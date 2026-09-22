# Plan Review

Skill para revisar planos de implementação antes de escrever código: verifica
se o plano resolve o problema certo, se a solução cabe no projeto e se cada
tarefa tem instruções e critérios de aceite suficientes para execução.

## Instalação

Requer Node.js e npm. Depois que este repositório estiver disponível no GitHub,
substitua `OWNER/REPO` pelo caminho dele:

```sh
npx skills add OWNER/REPO --skill plan-review
```

O CLI permite escolher o agente e o escopo da instalação.

## Uso

Peça ao agente para revisar um plano e forneça o arquivo ou o texto:

```text
Use plan-review para revisar plans/exemplo/plan.md.
```

A revisão é somente leitura e apresenta os achados, as evidências e um
veredito. As instruções completas estão em [SKILL.md](SKILL.md).

## Publicação no skills.sh

Mantenha `SKILL.md` na raiz, com `name` e `description` no frontmatter YAML,
e publique os arquivos versionados em um repositório público no GitHub.

Valide a descoberta local sem instalar:

```sh
DISABLE_TELEMETRY=1 npx skills add . --list
```

Depois de publicar, confira a descoberta remota:

```sh
npx skills add OWNER/REPO --list
```

A instalação usa o comando da seção Instalação. Segundo a documentação, a
listagem no skills.sh acontece automaticamente pela telemetria das instalações;
a publicação no GitHub e a aparição no catálogo são etapas distintas.

Documentação: [skills.sh](https://www.skills.sh/docs),
[FAQ](https://www.skills.sh/docs/faq) e
[CLI e formato das skills](https://github.com/vercel-labs/skills).

## Avaliações locais

`evals/` contém dados de projetos privados e é ignorado pelo Git.
Mantenha fixtures e resultados de avaliação privados nesse diretório.
