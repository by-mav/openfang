# DIRECTOR BRAIN — Video Director Knowledge Base

## 1. REFERÊNCIAS DE ADS QUE FUNCIONAM

### Nicho Saúde/Estudo (CONCORRENTES DIRETOS DO VITAAI)
| App | O que funciona | Formato | Duração |
|-----|---------------|---------|---------|
| **Duolingo** | Mascot em cenários absurdos. Nativo TikTok, humor > venda. Participa de trends | UGC/meme-style | 15-30s |
| **Headspace** | Animações calmas, paleta wellness. YouTube com influencers longos | Problem-solution animado + influencer | 15-30s ads, 10-20min YT |
| **Calm** | Targeting contextual (horário commute). Celebridades (LeBron). Premium feel | Problem-solution polished | 15-30s social, 30-60s YT |
| **Quizlet** | Conteúdo de aluno real. "Study with me". Stress de prova relatável | UGC student-created | 15-30s TikTok |
| **Anki** | Med students mostrando resultados. "How I passed Step 1". Dados de retenção | UGC creator testimonial + demo | 30-60s TikTok, 5-15min YT |

**LIÇÃO PRO VITAAI:** Anki é o concorrente mais próximo. Alunos de medicina que mostram RESULTADOS (notas subindo) e ROTINA (usando o app) convertem. Focar em UGC de estudante real + problem-solution curto.

### Nicho Fintech (CONCORRENTES DO PIXIO)
| App | O que funciona | Formato | Duração |
|-----|---------------|---------|---------|
| **Nubank** | "Nu World" — app na vida real. Storytelling emocional, transformação financeira | Emotional storytelling + problem-solution | 30-60s brand, 15-30s social |
| **Cash App** | Meme-adjacent, influencers TikTok. Cool factor. Cultura música/esporte | UGC + Spark Ads | 15-30s TikTok |
| **Revolut** | "Your Way In" — desafia estereótipos, underdog. Clean, simples | Problem-solution + brand awareness | 30-60s brand, 15s retargeting |
| **Chime** | "Get paid early" como herói. Wayne Brady: 368M impressions, 115M views | Celebrity + demo hybrid | 15-30s social |
| **Robinhood** | "We are all investors" — diversidade, democratizar. Animações in-app (confetti) | Aspiracional + demo com overlays | 30-60s brand, 15-30s social |

**LIÇÃO PRO PIXIO:** Nubank é a referência #1. Storytelling emocional + vida real. Mostrar transformação: "bagunça financeira → Pixio → controle total". Open Finance como diferencial (conecta TODOS os bancos).

---

## 2. TTS — VOICEOVER AUTOMÁTICO EM PT-BR

### Recomendação BYMAV
| Uso | Provider | Custo | Por que |
|-----|----------|-------|---------|
| **Hero videos** (marketing principal) | ElevenLabs | $5/mo starter (30k chars) | Melhor qualidade emocional. API `/with-timestamps` retorna timing por CARACTERE — perfeito pra sync Remotion |
| **Bulk content** (volume alto) | OpenAI TTS | $15/1M chars | 12x mais barato que ElevenLabs. Qualidade boa pra pt-BR |
| **Fallback** | Google Cloud TTS | $16/1M Neural | Enterprise, 300+ vozes, muito confiável |

### Pipeline TTS → Remotion
```
1. Script (texto) → ElevenLabs API /convert-with-timestamps
2. Retorna: audio.mp3 + alignment.json (timing por caractere)
3. Converter timestamps pra frames: frame = seconds * fps
4. Remotion <Audio src="narration.mp3" /> + <Sequence> por segmento
5. Captions animadas sincronizadas com a narração
```

### ElevenLabs API (endpoint exato)
```
POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}/with-timestamps
Body: { "text": "Narração aqui", "model_id": "eleven_multilingual_v2" }
Response: { audio: base64, alignment: { characters, character_start_times_seconds, character_end_times_seconds } }
```

### Remotion TTS Template
Azure template oficial: https://www.remotion.dev/templates/tts
ElevenLabs example: https://github.com/FelippeChemello/Remotion-TTS-Example

---

## 3. SCREEN RECORDING AUTOMATIZADO

### Pipeline pra demos de produto
```
1. Puppeteer/Playwright headless → navega no app → executa ações → captura frames
2. puppeteer-screen-recorder (npm) → gera video.mp4 do fluxo
3. Importar no Remotion como <Video src="demo-recording.mp4" />
4. Remotion adiciona: overlays, zoom, cursor highlight, text callouts, transições
5. Combinar com TTS narração
6. npx remotion render → MP4 final
```

### Puppeteer screencast (nativo)
```javascript
const recorder = await page.screencast({ path: 'demo.webm' });
// ... navegar, clicar, mostrar features
await recorder.stop();
```

### Playwright (alternativa)
```javascript
const context = await browser.newContext({ recordVideo: { dir: './videos' } });
// ... ações
await context.close(); // video salvo automaticamente
```

### ffmpeg X11 capture (nosso Xvfb)
```bash
ffmpeg -f x11grab -r 30 -s 1920x1080 -i :99 -c:v libx264 output.mp4
```
Funciona com o Brave dos agentes no Xvfb :99!

---

## 4. WORKFLOW COMPLETO: IDEA → VIDEO PRONTO

### Fluxo automatizado (2-5 min por vídeo)
```
1. DIRECTOR escreve script (usando AIDA/PAS/BAB)
2. TTS gera áudio + timestamps (ElevenLabs ou OpenAI)
3. Puppeteer captura screen recording do app (headless)
4. DIRECTOR monta composição Remotion:
   - <Sequence> Hook (0-3s): texto bold + animação impactante
   - <Sequence> Body (3-20s): screen recording + overlays + narração
   - <Sequence> CTA (20-25s): logo + call to action
   - <Audio> narração sincronizada
   - <Captions> legendas TikTok-style
5. Renderiza em múltiplos formatos:
   - 9:16 (1080x1920) → TikTok, Reels, Shorts
   - 4:5 (1080x1350) → Instagram Feed
   - 16:9 (1920x1080) → YouTube
6. Gera 3-5 variações de hook (swap primeiros 3s)
7. Export thumbnails dos melhores frames
```

### Variações automáticas por código
```typescript
const hooks = [
  "Reprovando em medicina? O Vita resolve.",
  "Estudei 2h por dia e passei em tudo. Te mostro como.",
  "Seu professor não te ensina a estudar. O Vita ensina.",
  "3 alunos de medicina. 1 usa IA. Adivinha quem passou.",
  "Para de ler resumo. Começa a estudar certo."
];
// Renderizar 5 variações automaticamente
for (const hook of hooks) {
  await renderMedia({ composition: 'VitaAd', inputProps: { hook } });
}
```

---

## 5. TEMPLATES REMOTION NECESSÁRIOS (criar primeiro)

### Template 1: Problem-Solution (PRIORIDADE #1)
- Hook (3s): texto bold com dor do aluno
- Problema (5s): tela confusa, stress
- Solução (10s): demo do app com overlays
- CTA (5s): logo + "Baixe grátis"
- Total: 23s (sweet spot TikTok)

### Template 2: UGC-Style Demo
- Hook (3s): "como eu estudo medicina"
- Walkthrough (20s): screen recording com narração
- Resultado (5s): notas/aprovação
- CTA (3s): link
- Total: 31s

### Template 3: Before/After
- Before (5s): planilha/caderno bagunçado
- Transição (2s): wipe/morph
- After (10s): app organizado, dados claros
- CTA (5s): "Transforma teu estudo"
- Total: 22s

### Template 4: Estatística Impactante
- Hook (3s): "84% dos alunos que usam IA passam"
- Dados (10s): gráficos animados, números subindo
- Demo (10s): app em ação
- CTA (5s): "Faça parte dos 84%"
- Total: 28s

---

## 6. CHECKLIST PRÉ-PRODUÇÃO

Antes de QUALQUER vídeo:
- [ ] Concorrente analisado (Meta Ad Library ou TikTok Creative Center)
- [ ] Script escrito usando framework (AIDA/PAS/BAB/Hook-Body-CTA)
- [ ] 3-5 variações de hook definidas
- [ ] Formato definido por plataforma (9:16, 4:5, 16:9)
- [ ] Duração definida (21-34s TikTok, 15-30s Reels, 30-60s YT)
- [ ] Narração: TTS API escolhida (ElevenLabs hero / OpenAI bulk)
- [ ] Captions: estilo definido (TikTok-style word highlight)
- [ ] Thumbnail: frame de hook otimizado
- [ ] Música: trending audio ou royalty-free
- [ ] CTA: claro, no final, 3-5s
