# ✨ Contador de Palavras — Guia Didático e Técnico

[![Teste online agora!](https://img.shields.io/badge/Teste_online_agora-🚀-brightgreen)](https://carlosfreires.github.io/contador-de-palavras-webpage/)

> **Descrição:**
> Ferramenta pequena e didática construída em **HTML + CSS + JavaScript puro** que analisa um texto colado ou digitado no navegador, calculando o total de palavras, removendo stopwords (palavras muito comuns) e exibindo a frequência das palavras relevantes. Todo o projeto está contido em um único arquivo e escrito em menos de 90 linhas de código.

---

## Sumário

1. Objetivo
2. Funcionalidades principais
3. Estrutura do repositório
4. Requisitos
5. Como executar localmente
6. Explicação detalhada do código
   - HTML
   - CSS
   - JavaScript (passo a passo)
7. Boas práticas, casos de teste e validação
8. Sugestões de extensões e atividades para estudantes
9. Anexos: `index.html` (arquivo completo e pronto)
10. Licença

---

## 1. Objetivo

Este README tem propósito duplo:

- **Pedagógico:** servir como material de estudos.
- **Técnico:** documentar a implementação, as decisões de projeto, limitações e caminhos de evolução.

O conteúdo foi pensado para ser compreendido por qualquer pessoa com noções básicas de HTML, CSS e JavaScript.

---

## 2. Funcionalidades principais

- Receber texto via `textarea`.
- Normalizar e limpar o texto (remoção de pontuação e caracteres especiais).
- Contabilizar o total de palavras.
- Filtrar *stopwords* (lista configurável de palavras muito comuns).
- Calcular frequência das palavras relevantes e ordenar do mais ao menos frequente.
- Exibir os resultados de forma simples, responsiva e acessível.

---

## 3. Estrutura do repositório

```bash
/
├── index.html       # Implementação completa (HTML + CSS + JS)
└── README.md        # Documentação didática (este arquivo)
```

---

## 4. Requisitos

- Navegador moderno (Chrome, Edge, Firefox, Safari) com suporte a ECMAScript moderno e `Intl`/Unicode RegExp (`\p{L}`).
- JavaScript habilitado.

> Observação técnica: o uso de `\p{L}` em expressões regulares exige motores JS modernos. Em ambientes antigos, pode ser necessário adaptar a limpeza de texto.

---

## 5. Como executar localmente

1. Clone ou baixe o repositório:

```bash
git clone https://github.com/carlosfreires/contador-de-palavras-webpage.git
cd contador-de-palavras-webpage
```

2. Abra o arquivo `index.html` em um navegador:

- Windows: `start index.html`
- macOS: `open index.html`
- Linux (GUI): `xdg-open index.html`

Nenhum servidor é necessário - é um *static single-file*.

---

## 6. Explicação detalhada do código

A seguir, a estrutura lógica do `index.html` explicada em detalhe. O arquivo completo está no final.

### 6.1 HTML - estrutura semântica

```html
<main>
  <h1>Contador de Palavras</h1>
  <textarea id="entradaTexto" placeholder="Digite ou cole o texto aqui..."></textarea>
  <button id="botaoAnalisar">Analisar Texto</button>
  <div id="resultado" aria-live="polite"></div>
</main>
```

Pontos-chave:

- Uso de `<main>` para conteúdo principal (melhora a semântica e acessibilidade).
- `aria-live="polite"` em `#resultado` informa leitores de tela sobre atualizações da área de resultados.
- Inputs minimalistas; a interface é deliberadamente simples para foco didático.

### 6.2 CSS - design e usabilidade

As decisões de estilo visam clareza, legibilidade e responsividade:

- Variáveis CSS (`:root`) para cores e raio de borda.
- `display: flex` na `body` para centralizar vertical/horizontalmente.
- `max-height` com `overflow-y: auto` em `#resultado` para comportar muitas linhas.

### 6.3 JavaScript - passo a passo (lógica)

A lógica central do sistema está organizada em etapas claras - tratar entrada, normalizar, tokenizar, filtrar, contar e apresentar.

#### Seleção de elementos

```js
const entradaTexto = document.getElementById('entradaTexto');
const botaoAnalisar = document.getElementById('botaoAnalisar');
const resultado = document.getElementById('resultado');
```

#### Stopwords (palavras comuns)

A lista a seguir é um ponto de partida - ela pode (e deve) ser estendida conforme o domínio do texto.

```js
const palavrasComuns = new Set([
  'a','o','e','de','do','da','em','que','por','para','com',
  'um','uma','sao','são','nao','não','mas','foi','ser','os','as','na','no'
]);
```

> Nota técnica: inclui formas sem e com acento para maior robustez - outra alternativa é normalizar (remover diacríticos) antes da comparação.

#### Evento: clicar em "Analisar Texto"

- Valida entrada vazia.
- Normaliza o texto (minúsculas + remoção de acentos e caracteres não-alfabéticos).
- Separa palavras por espaços consecutivos (tokenização simples).
- Filtra stopwords.
- Calcula frequência com `Map`.
- Ordena por frequência decrescente e apresenta no DOM.

Trecho principal (explicado):

```js
botaoAnalisar.addEventListener('click', () => {
  const textoOriginal = entradaTexto.value;
  if (!textoOriginal.trim()) {
    resultado.innerHTML = '<p>Por favor, insira algum texto.</p>';
    return;
  }

  // 1) Normalizar caixa e remover diacríticos (acentos)
  const textoNormalizado = textoOriginal
    .toLowerCase()
    .normalize('NFD') // separa letras de seus diacríticos
    .replace(/[̀-ͯ]/g, ''); // remove os diacríticos

  // 2) Remover tudo que não for letra ou espaço (preservando letras Unicode)
  const textoLimpo = textoNormalizado.replace(/[^\p{L}\s]/gu, '').trim();

  // 3) Tokenizar por espaços
  const todasAsPalavras = textoLimpo.split(/\s+/).filter(Boolean);
  const totalPalavras = todasAsPalavras.length;

  // 4) Filtrar stopwords
  const palavrasFiltradas = todasAsPalavras.filter(p => !palavrasComuns.has(p));

  // 5) Contar frequências
  const frequencia = new Map();
  for (const palavra of palavrasFiltradas) {
    frequencia.set(palavra, (frequencia.get(palavra) || 0) + 1);
  }

  // 6) Ordenar por frequência
  const ordenadas = Array.from(frequencia.entries()).sort((a, b) => b[1] - a[1]);

  // 7) Construir saída (top 100 por padrão)
  let saidaHTML = `<p><strong>Total de palavras:</strong> ${totalPalavras}</p>`;

  if (ordenadas.length > 0) {
    saidaHTML += '<hr>';
    saidaHTML += ordenadas.slice(0, 100).map(([palavra, contagem]) => `
      <p><strong>${palavra}</strong>: ${contagem}</p>
    `).join('');
  } else if (totalPalavras > 0) {
    saidaHTML += '<p>(Todas as palavras foram filtradas por serem muito comuns.)</p>';
  } else {
    saidaHTML = '<p>Nenhuma palavra válida foi encontrada.</p>';
  }

  resultado.innerHTML = saidaHTML;
});
```

#### Observações sobre segurança

- Como o aplicativo roda localmente no navegador e a entrada vem do próprio usuário, o risco é baixo. Ainda assim, quando for exibir resultados construídos a partir de texto livre, prefira criar nós DOM com `textContent` ou escapar trechos para evitar possíveis injeções, se o código evoluir para aceitar conteúdo remoto.

---

## 7. Boas práticas, casos de teste e validação

### Casos de teste sugeridos

1. Texto vazio -> mensagem solicitando entrada.
2. Texto com várias pontuações (vírgulas, pontos, emojis) -> garantir que a limpeza remove os símbolos.
3. Palavras com acentuação (ação, açao) -> verificar se a normalização agrupa adequadamente.
4. Texto com muitos espaços e quebras de linha -> tokenização deve ignorar espaços extras.
5. Texto em outro idioma (inglês/português misturados) -> a tokenização continua válida; porém a lista de stopwords é idioma-dependente.

### Métricas e expectativas

- **Precisão da contagem:** para textos simples (sem símbolos especializados), a contagem deverá ser precisa.
- **Limitações:** a tokenização é *white-space splitting* - não trata casos complexos como contrações ("don't"), hífens compostos específicos ou segmentação por idioma avançada.

---

## 8. Sugestões de extensões (atividades para estudantes)

1. Adicionar campo para customizar stopwords na interface.
2. Exportar resultado para CSV.
3. Mostrar uma nuvem de palavras (word cloud).
4. Permitir análise em tempo real (com `input` + debounce).
5. Incluir opção para ignorar palavras muito curtas (ex.: < 3 caracteres).
6. Adicionar testes unitários com `Jest` (testar funções de limpeza/tokenização/contagem).
7. Internacionalizar: carregar stopwords por idioma (JSON externo).

Cada uma dessas atividades é um bom exercício prático para consolidar conceitos de DOM, eventos e manipulação de strings.

---

## 9. `index.html`

> Abaixo está o arquivo `index.html` com a implementação corrigida, comentada e pronta para uso. Copie e cole em um arquivo `index.html` e abra no navegador.

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contador de Palavras</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --cor-principal: #4361ee;
      --fundo: #f8f9fa;
      --borda-arredondada: 12px;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: "Poppins", sans-serif; background: var(--fundo); display: flex; justify-content: center; align-items: center; height: 100vh; }
    main { background: var(--cor-principal); color: #fff; padding: 2rem; border-radius: var(--borda-arredondada); box-shadow: 0 4px 16px rgba(0,0,0,0.15); width: min(90%, 600px); }
    textarea, button { width: 100%; margin-top: 1rem; padding: .75rem; border: none; border-radius: var(--borda-arredondada); font-size: 1rem; }
    textarea { resize: none; height: 120px; }
    button { background: #fff; color: var(--cor-principal); font-weight: 600; cursor: pointer; transition: .3s; }
    button:hover { background: #e9ecef; }
    #resultado { margin-top: 1rem; background: #fff; color: #333; padding: 1rem; border-radius: var(--borda-arredondada); max-height: 200px; overflow-y: auto; }
    hr { border: none; height: 1px; background: rgba(0,0,0,0.06); margin: 0.75rem 0; }
  </style>
</head>
<body>
  <main>
    <h1>Contador de Palavras</h1>
    <textarea id="entradaTexto" placeholder="Digite ou cole o texto aqui..."></textarea>
    <button id="botaoAnalisar">Analisar Texto</button>
    <div id="resultado" aria-live="polite"></div>
  </main>

  <script>
    const entradaTexto = document.getElementById('entradaTexto');
    const botaoAnalisar = document.getElementById('botaoAnalisar');
    const resultado = document.getElementById('resultado');

    // Lista inicial de stopwords (exemplo). Personalize conforme necessidade.
    const palavrasComuns = new Set([
      'a','o','e','de','do','da','em','que','por','para','com',
      'um','uma','sao','são','nao','não','mas','foi','ser','os','as','na','no'
    ]);

    botaoAnalisar.addEventListener('click', () => {
      const textoOriginal = entradaTexto.value;

      if (!textoOriginal.trim()) {
        resultado.innerHTML = '<p>Por favor, insira algum texto.</p>';
        return;
      }

      // Normaliza caixa e remove diacríticos para agrupar forma com/sem acento
      const textoNormalizado = textoOriginal
        .toLowerCase()
        .normalize('NFD')
        .replace(/[\u0300-\u036f]/g, '');

      // Remove tudo que não seja letra ou espaço (mantém letras Unicode)
      // exige suporte a \p{L} (ES2018+)
      const textoLimpo = textoNormalizado.replace(/[^\p{L}\s]/gu, '').trim();

      // Tokeniza por espaços em branco (inclui quebras de linha)
      const todasAsPalavras = textoLimpo.split(/\s+/).filter(Boolean);
      const totalPalavras = todasAsPalavras.length;

      // Filtra stopwords
      const palavrasFiltradas = todasAsPalavras.filter(p => !palavrasComuns.has(p));

      // Conta frequência
      const frequencia = new Map();
      for (const palavra of palavrasFiltradas) {
        frequencia.set(palavra, (frequencia.get(palavra) || 0) + 1);
      }

      // Ordena por frequência (decrescente)
      const ordenadas = Array.from(frequencia.entries()).sort((a, b) => b[1] - a[1]);

      // Monta saída (top 100)
      let saidaHTML = `<p><strong>Total de palavras:</strong> ${totalPalavras}</p>`;

      if (ordenadas.length > 0) {
        saidaHTML += '<hr>';
        saidaHTML += ordenadas.slice(0, 100).map(([palavra, contagem]) => `\n  <p><strong>${palavra}</strong>: ${contagem}</p>`).join('');
      } else if (totalPalavras > 0) {
        saidaHTML += '<p>(Todas as palavras foram filtradas por serem muito comuns.)</p>';
      } else {
        saidaHTML = '<p>Nenhuma palavra válida foi encontrada.</p>';
      }

      resultado.innerHTML = saidaHTML;
    });
  </script>
</body>
</html>
```

---

## 10. Licença

Uso livre.

---

### Observação

Este repositório foi criado com o objetivo de estimular a evolução contínua e o aprendizado colaborativo. As sugestões de extensão funcionam como mini-projetos, permitindo aos participantes exercitar seus conhecimentos de forma prática e incentivar a troca de experiências.
