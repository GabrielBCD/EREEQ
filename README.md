# 🔍 Auditoria Completa — I-EREEQ-CO (React + Vite)

---

## 🔴 CRÍTICO

### 1. `<button>` aninhado dentro de `<a>` — HTML inválido e inacessível

**Arquivo:** `src/components/button/index.jsx`

```jsx
// ❌ PROBLEMÁTICO: `<button>` dentro de `<a>` é HTML inválido (spec W3C)
export function Button({ children, link }) {
    return (
        <a href={link}>
            <button className='button'>
                {children}
            </button>
        </a>
    )
}
```

Isso cria dois elementos interativos aninhados, confunde leitores de tela, e é inválido segundo a especificação HTML5. A solução é usar apenas um elemento:

```jsx
// ✅ CORRETO: <a> estilizado como botão, ou <button> com onClick
export function Button({ children, href }) {
    return (
        <a href={href} className='button' target="_blank" rel="noopener noreferrer">
            {children}
        </a>
    )
}
```

> Também renomeie a prop de `link` para `href` — é o nome canônico do atributo HTML.

---

### 2. Hierarquia de headings quebrada (`<h1>` múltiplos e fora de ordem)

**Arquivos:** `Banner/index.jsx`, `ObjectivesAndActivities/index.jsx`, `TopicsDiscussed/index.jsx`, `EventImportance/index.jsx`, `Baseboard/index.jsx`

A página renderiza este fluxo de headings:

| Componente | Heading atual | Correto |
|---|---|---|
| `Banner` | dois `<h1>` | um `<h1>` |
| `AboutEvent` | `<h2>` | ✅ `<h2>` |
| `ObjectivesAndActivities` | `<h1>` | `<h2>` |
| `TopicsDiscussed` | `<h1>` | `<h2>` |
| `EventImportance` | `<h1>` | `<h2>` |
| `Baseboard` | `<h1>` texto de copyright | `<p>` ou `<small>` |

```jsx
// ❌ Banner/index.jsx
<h1>I Encontro Regional de Educação Escolar</h1>
<h1>Quilombola do Centro-Oeste</h1>

// ✅ Correto
<h1>I Encontro Regional de Educação Escolar Quilombola do Centro-Oeste</h1>
```

```jsx
// ❌ Baseboard/index.jsx — copyright em <h1> é completamente errado
<h1>© 2025 – I Encontro Regional...</h1>

// ✅ Correto
<p>© 2025 – I Encontro Regional...</p>
```

---

### 3. Ausência de atributos ARIA essenciais na navegação

**Arquivo:** `src/components/Cabecalho/index.jsx`

O `<nav>` não tem label e os links ativos não comunicam o estado para tecnologias assistivas:

```jsx
// ❌ Problemático
<div className='header'>
    <div className='logo'>
        <NavLink to="/"><h1>I EREEQ-CO</h1></NavLink>
    </div>
    <div>
        <nav className='menu'>
            <NavLink to="/" >Sobre o Evento</NavLink>
```

```jsx
// ✅ Correto
<div className='header' role="banner">
    <div className='logo'>
        {/* h1 dentro de NavLink cria foco confuso — use span ou p estilizado */}
        <NavLink to="/" aria-label="Ir para a página inicial">
            <span className='logo-text'>I EREEQ-CO</span>
        </NavLink>
    </div>
    <nav className='menu' aria-label="Navegação principal">
        <NavLink to="/" aria-current={isActive ? "page" : undefined}>
            Sobre o Evento
        </NavLink>
    </nav>
</div>
```

O `NavLink` do React Router já fornece a classe `active` — basta mapear isso para o `aria-current`:

```jsx
<NavLink
    to="/"
    className={({ isActive }) => isActive ? 'active' : ''}
    aria-current={({ isActive }) => isActive ? 'page' : undefined}
>
    Sobre o Evento
</NavLink>
```

---

### 4. `<h1>` com logo de navegação — Semântica incorreta

**Arquivo:** `src/components/Cabecalho/index.jsx`

```jsx
// ❌ h1 não deve ser reutilizado para o logo do header
<NavLink to="/"><h1>I EREEQ-CO</h1></NavLink>

// ✅ Use a tag correta; estilize via CSS
<NavLink to="/" aria-label="Página inicial – I EREEQ-CO">
    <span className='logo-text'>I EREEQ-CO</span>
</NavLink>
```

---

### 5. `lang="en"` em um site 100% em português — crítico para acessibilidade

**Arquivo:** `index.html`

```html
<!-- ❌ Idioma declarado como "en" mas o conteúdo é todo em português -->
<html lang="en">
```

```html
<!-- ✅ Correto -->
<html lang="pt-BR">
```

Leitores de tela usam o atributo `lang` para selecionar o idioma de pronúncia. Deixar `"en"` faz com que o conteúdo em português seja lido com pronúncia inglesa.

---

## 🟡 MELHORIA

### 6. Rotas sem Lazy Loading — bundle desnecessariamente grande

**Arquivo:** `src/App.jsx`

```jsx
// ❌ Tudo carregado imediatamente no bundle inicial
import HomePage from './pages/HomePage'
import LetterPage from './pages/LetterPage'
import ParticipantsPage from './pages/ParticipantsPage'
```

```jsx
// ✅ Code splitting por rota com React.lazy + Suspense
import { lazy, Suspense } from 'react'

const HomePage = lazy(() => import('./pages/HomePage'))
const LetterPage = lazy(() => import('./pages/LetterPage'))
const ParticipantsPage = lazy(() => import('./pages/ParticipantsPage'))

function App() {
  return (
    <Router>
      <Suspense fallback={<div aria-busy="true">Carregando...</div>}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/letter" element={<LetterPage />} />
          <Route path="/participants" element={<ParticipantsPage />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </Suspense>
    </Router>
  )
}
```

---

### 7. Páginas vazias sem conteúdo (`LetterPage` e `ParticipantsPage`)

**Arquivos:** `src/pages/LetterPage.jsx`, `src/pages/ParticipantsPage.jsx`

Ambas as páginas existem nas rotas mas não possuem `<main>`:

```jsx
// ❌ Página sem conteúdo — estrutura incompleta, falta <main>
function LetterPage() {
  return (
    <>
      <header><Cabecalho/></header>
      <footer><Baseboard /></footer>
    </>
  )
}
```

```jsx
// ✅ Estrutura mínima com placeholder semântico
function LetterPage() {
  return (
    <>
      <header><Cabecalho/></header>
      <main>
        <section aria-label="Carta do I EREEQ-CO">
          {/* Conteúdo a implementar */}
        </section>
      </main>
      <footer><Baseboard /></footer>
    </>
  )
}
```

---

### 8. Imports de `App.css` vazio em todos os arquivos de página

**Arquivos:** `src/App.jsx`, `src/pages/HomePage.jsx`, `src/pages/LetterPage.jsx`, `src/pages/ParticipantsPage.jsx`

```jsx
// ❌ App.css está completamente vazio — importar é ruído desnecessário
import '../App.css'  // ou './App.css' no App.jsx
```

Remova essa linha dos 4 arquivos. O arquivo `App.css` deveria ser deletado ou preenchido com estilos globais relevantes.

---

### 9. Caminhos de URLs inconsistentes (case)

**Arquivo:** `src/App.jsx`

```jsx
// ❌ Mistura de convenções: "/" é lowercase, mas "/Letter" e "/Participants" são PascalCase
<Route path="/Letter" element={<LetterPage/>} />
<Route path="/Participants" element={<ParticipantsPage/>} />
```

URLs devem ser sempre **lowercase** por convenção web e SEO:

```jsx
// ✅ Correto
<Route path="/letter" element={<LetterPage/>} />
<Route path="/participants" element={<ParticipantsPage/>} />
```

> Atualize também os `<NavLink>` em `Cabecalho/index.jsx` para `/letter` e `/participants`.

---

### 10. Página 404 sem componente dedicado

**Arquivo:** `src/App.jsx`

```jsx
// ❌ Bare <h1> como 404 — sem layout, sem navegação de volta
<Route path='*' element={<h1>Not Found</h1>} />
```

```jsx
// ✅ Crie src/pages/NotFoundPage.jsx
function NotFoundPage() {
  return (
    <>
      <header><Cabecalho /></header>
      <main>
        <section aria-label="Página não encontrada">
          <h1>Página não encontrada</h1>
          <p>O endereço que você acessou não existe.</p>
          <a href="/">Voltar para a página inicial</a>
        </section>
      </main>
      <footer><Baseboard /></footer>
    </>
  )
}
```

---

### 11. Alturas fixas em seções quebram responsividade

**Arquivos:** `aboutEvent.estilos.css`, `objectivesAndActivities.estilos.css`

```css
/* ❌ Quebra quando o conteúdo cresce ou em telas menores */
.aboutEvent { height: 500px; }
.objectivesAndActivities { height: 550px; }
```

```css
/* ✅ Correto: deixa o conteúdo definir a altura */
.aboutEvent { min-height: 500px; }
.objectivesAndActivities { min-height: 550px; }
```

---

### 12. Caminho de imagem relativo no CSS quebrará em produção

**Arquivo:** `banner.estilos.css`

```css
/* ❌ Caminho relativo ao CSS — pode falhar dependendo do base path do Vite */
.banner {
  background-image: url('../../../public/banner.jpg');
}
```

```css
/* ✅ Caminho absoluto a partir da raiz pública (funciona sempre no Vite) */
.banner {
  background-image: url('/banner.jpg');
}
```

---

### 13. Convenção de nomenclatura de arquivos CSS inconsistente

O projeto mistura três padrões diferentes:

| Arquivo | Padrão |
|---|---|
| `cabecalho.estilos.css` | kebab-case, PT |
| `Baseboard.estilos.css` | PascalCase, EN |
| `aboutEvent.estilos.css` | camelCase, EN |

**Recomendação:** Padronize para `kebab-case.module.css` ou `NomeDoComponente.module.css` e adote **CSS Modules** para evitar conflitos de classe globais. Exemplo: a classe `.container` em `objectivesAndActivities.estilos.css` é perigosamente genérica e pode colidir com outros estilos.

---

### 14. Nome do componente em português (`Cabecalho`) diverge dos demais

O projeto usa inglês em tudo (`Banner`, `Baseboard`, `TopicsDiscussed`) exceto `Cabecalho`. Padronize para `Header` ou `Navbar` para consistência com o restante da codebase.

---

## 🎨 SUGESTÃO ESTÉTICA

### 15. Variáveis de ambiente para URLs hardcoded

**Arquivo:** `src/components/Banner/index.jsx`

```jsx
// ❌ URL hardcoded
<Button link="https://www.youtube.com/">Conheça o Evento</Button>
```

```jsx
// ✅ Crie .env na raiz:
// VITE_YOUTUBE_URL=https://www.youtube.com/channel/seu-canal

// No componente:
<Button href={import.meta.env.VITE_YOUTUBE_URL}>Conheça o Evento</Button>
```

---

### 16. Ausência de `<meta name="description">` no `index.html`

```html
<!-- ✅ Adicione ao <head> para SEO e compartilhamento social -->
<meta name="description" content="I Encontro Regional de Educação Escolar Quilombola do Centro-Oeste — espaço de diálogo, valorização cultural e fortalecimento da Educação Escolar Quilombola." />
<meta property="og:title" content="I EREEQ-CO" />
<meta property="og:description" content="I Encontro Regional de Educação Escolar Quilombola do Centro-Oeste." />
```

---

### 17. Estrutura de pastas: diretórios faltantes para escalabilidade

A estrutura atual é funcional mas não escalável:

```
src/
├── components/   ✅
├── pages/        ✅
├── hooks/        ❌ faltando
├── utils/        ❌ faltando
├── constants/    ❌ faltando (para strings, URLs, config)
└── assets/       ✅
```

Quando o projeto crescer (filtros, busca de participantes, formulários), a ausência de `hooks/` e `utils/` levará a código duplicado ou lógica misturada na UI.

---

## 📊 Resumo por Criticidade

| # | Problema | Criticidade | Arquivo(s) |
|---|---|---|---|
| 1 | `<button>` dentro de `<a>` | 🔴 Crítico | `button/index.jsx` |
| 2 | Hierarquia de headings quebrada | 🔴 Crítico | Múltiplos |
| 3 | Falta de ARIA na navegação | 🔴 Crítico | `Cabecalho/index.jsx` |
| 4 | `<h1>` no logo do header | 🔴 Crítico | `Cabecalho/index.jsx` |
| 5 | `lang="en"` no HTML em português | 🔴 Crítico | `index.html` |
| 6 | Sem Lazy Loading nas rotas | 🟡 Melhoria | `App.jsx` |
| 7 | Páginas vazias sem `<main>` | 🟡 Melhoria | `LetterPage`, `ParticipantsPage` |
| 8 | Import de `App.css` vazio | 🟡 Melhoria | 4 arquivos |
| 9 | URLs com PascalCase | 🟡 Melhoria | `App.jsx`, `Cabecalho` |
| 10 | 404 sem componente dedicado | 🟡 Melhoria | `App.jsx` |
| 11 | Alturas fixas quebrando responsivo | 🟡 Melhoria | 2 CSS files |
| 12 | Caminho de imagem relativo no CSS | 🟡 Melhoria | `banner.estilos.css` |
| 13 | Nomenclatura CSS inconsistente | 🟡 Melhoria | Todos os CSS |
| 14 | Nome de componente em PT (`Cabecalho`) | 🟡 Melhoria | `Cabecalho/` |
| 15 | URLs hardcoded (sem `.env`) | 🎨 Estética | `Banner/index.jsx` |
| 16 | Meta description ausente | 🎨 Estética | `index.html` |
| 17 | Falta de `hooks/`, `utils/` | 🎨 Estética | Estrutura de pastas |

---

## ✅ O Que Está Bem

- **Zero problemas com hooks React** — componentes são puramente apresentacionais, sem `useEffect`/`useMemo` mal utilizados.
- **Dependências atuais** — React 19, Vite 7, React Router 7, todos na versão mais recente.
- **ESLint configurado** corretamente com o plugin `react-hooks`.
- **StrictMode ativado** em `main.jsx`.
- **`<main>` presente** em `HomePage.jsx` com estrutura semântica.
- **`alt` text** corretamente definido na imagem de `TopicsDiscussed`.
- **CSS colocado junto ao componente** (co-location pattern).
