# 🐛 CVE — Vulnerabilidades Reportadas e Analisadas

> CVEs identificadas, reportadas e publicadas em meu nome.  
> Cada entrada inclui análise técnica completa, Prova de Conceito (PoC) e orientações de mitigação.

[![MITRE CVE](https://img.shields.io/badge/MITRE-CVE-FF0000?style=flat-square)](https://cve.mitre.org)
[![NVD NIST](https://img.shields.io/badge/NVD-NIST-003087?style=flat-square)](https://nvd.nist.gov)
[![Reported by](https://img.shields.io/badge/Reported%20by-italoantunes-181717?style=flat-square&logo=github)](https://github.com/italoantunes)

---

## Sobre este repositório

As vulnerabilidades documentadas aqui foram **identificadas, reportadas e creditadas a mim** junto ao MITRE/NVD seguindo o processo de Responsible Disclosure — notificação privada ao fornecedor, prazo para correção e publicação coordenada após o patch.

Cada CVE passa por:

1. **Identificação** — descoberta da vulnerabilidade através de análise manual ou automatizada
2. **Documentação** — registro técnico detalhado com impacto, vetor de ataque e PoC
3. **Reporte responsável** — notificação ao fornecedor com prazo para correção
4. **Publicação** — divulgação coordenada após correção, com crédito no registro oficial do MITRE

> ⚠️ **Aviso:** Todo conteúdo é estritamente para fins educacionais e de pesquisa em segurança. Nunca utilize PoCs contra sistemas sem autorização explícita.

---

## CVEs Documentadas

| CVE | Vulnerabilidade | Severidade | Sistema |
|---|---|---|---|
| [CVE-2020-35735](#cve-2020-35735--clickjacking-no-vidyo-portal) | Clickjacking | MEDIUM | Vidyo Portal |

---

## CVE-2020-35735 — Clickjacking no Vidyo Portal

**🔗 NVD:** https://nvd.nist.gov/vuln/detail/CVE-2020-35735  
**📅 Publicada:** 2020  
**🎯 Sistema afetado:** Vidyo Portal (plataforma de videoconferência)  
**⚠️ Severidade:** MEDIUM  
**🔍 CWE:** CWE-1021 — Improper Restriction of Rendered UI Layers  
**👤 Descoberta e reportada por:** [Italo Antunes de Oliveira (@italoantunes)](https://github.com/italoantunes) — crédito registrado no MITRE/NVD

---

### O que é Clickjacking?

Imagine que você acessa um site aparentemente inofensivo — um banner de promoção, um botão de "clique aqui para ganhar um prêmio". Por baixo dessa página, invisível para você, existe outra página carregada dentro de um `<iframe>`: a página real de um sistema legítimo, como um portal de videoconferência.

Quando você clica no que parece ser o botão da promoção, na verdade está clicando em um botão do sistema real por baixo — sem saber.

Esse ataque se chama **Clickjacking** (sequestro de clique): o atacante "sequestra" seu clique redirecionando-o para uma ação em um sistema que você nem sabia que estava acessando.

```
O que o usuário vê:          O que realmente acontece:
┌─────────────────────┐      ┌─────────────────────┐  ← página do atacante
│                     │      │  Clique aqui! 🎁    │
│   [Clique aqui! 🎁] │      ├─────────────────────┤  ← Vidyo Portal (invisível, por baixo)
│                     │      │  [Iniciar reunião]  │  ← clique vai aqui
└─────────────────────┘      └─────────────────────┘
```

---

### A vulnerabilidade no Vidyo Portal

O **Vidyo** é uma plataforma corporativa de videoconferência usada por empresas, hospitais e instituições financeiras.

O portal de acesso não implementava o cabeçalho de segurança **`X-Frame-Options`** nem a diretiva **`Content-Security-Policy: frame-ancestors`** — ambos são mecanismos que instruem o browser a **não permitir** que a página seja carregada dentro de um `<iframe>` de outro domínio.

Sem essa proteção, qualquer site externo conseguia incorporar o portal do Vidyo de forma transparente (com `opacity: 0` ou `opacity: 0.5`) e sobrepor elementos visuais enganosos por cima.

---

### O que um atacante poderia fazer?

No contexto de um portal de videoconferência corporativo, as consequências práticas são:

**1. Iniciar ou encerrar reuniões sem consentimento**
O usuário clica no que parece ser um botão inofensivo e, na verdade, inicia uma reunião, encerra uma conferência em andamento ou aceita um participante externo.

**2. Vazar a câmera e o microfone**
Se a vítima for induzida a clicar em "Permitir" em um prompt de permissão de câmera/microfone do Vidyo — sem perceber que está interagindo com o portal — o atacante pode ativar captura de áudio e vídeo.

**3. Comprometer reuniões corporativas sensíveis**
Em empresas que usam o Vidyo para reuniões executivas, conselhos ou atendimentos médicos (telemedicina), o ataque poderia ser usado para adicionar participantes não autorizados ou gravar sessões confidenciais.

**4. Roubo de sessão por interação**
Induzir o usuário autenticado a executar ações dentro do portal — como alterar configurações, compartilhar tela ou confirmar convites — sem nenhum conhecimento.

---

### Prova de Conceito (PoC)

A PoC demonstra que o portal Vidyo pode ser carregado dentro de um `<iframe>` em uma página externa, confirmando a ausência dos headers de proteção:

```html
<!-- PoC simplificada — demonstra que o iframe carrega sem bloqueio -->
<iframe src="https://[vidyo-portal]/portal/"
        style="opacity: 0.5; width: 800px; height: 500px;">
</iframe>
```

Se o portal fosse protegido corretamente, o browser bloquearia o carregamento e exibiria um erro. Sem a proteção, o iframe carrega normalmente — confirmando a vulnerabilidade.

Os arquivos de PoC completos estão neste repositório:
- `Clickjacking.html` — ferramenta de teste interativa
- `Clickjacking-Vidyo_Portal_URI.html` — PoC específica para o portal Vidyo

---

### Como foi identificada

A vulnerabilidade foi identificada através de análise manual de headers HTTP da aplicação. A ausência dos cabeçalhos de segurança foi confirmada com ferramentas de inspeção de resposta HTTP, seguida da construção da PoC para demonstrar o impacto prático.

**Headers ausentes que permitiram o ataque:**

```http
# Nenhum destes estava presente na resposta do servidor:
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'
```

---

### Mitigação

A correção é simples — adicionar os headers HTTP na resposta do servidor:

**Opção 1 — Header HTTP (recomendado)**
```http
X-Frame-Options: DENY
```
ou, se precisar permitir embedding apenas do próprio domínio:
```http
X-Frame-Options: SAMEORIGIN
```

**Opção 2 — Content Security Policy (mais moderno)**
```http
Content-Security-Policy: frame-ancestors 'none'
```

**No servidor web:**

```nginx
# Nginx
add_header X-Frame-Options "DENY";
add_header Content-Security-Policy "frame-ancestors 'none'";
```

```apache
# Apache
Header always set X-Frame-Options "DENY"
Header always set Content-Security-Policy "frame-ancestors 'none'"
```

---

### Referências

- [NVD — CVE-2020-35735](https://nvd.nist.gov/vuln/detail/CVE-2020-35735)
- [OWASP — Clickjacking Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html)
- [CWE-1021 — Improper Restriction of Rendered UI Layers](https://cwe.mitre.org/data/definitions/1021.html)
- [MDN — X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

---

<div align="center">

*Mantido por [@italoantunes](https://github.com/italoantunes)*

</div>
