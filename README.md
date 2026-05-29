# OndaEsfera

Simulação interativa da **equação da onda em uma superfície esférica**, em tempo real no navegador, usando [Three.js](https://threejs.org/). Clique na esfera para excitar ondas que se propagam continuamente pela superfície — e, na versão completa, ouça a própria membrana vibrar.

## Demonstração

Os dois aplicativos são páginas HTML únicas, sem etapa de build. Sirva a pasta por HTTP (o áudio via `AudioWorklet` exige `http(s)`, não funciona em `file://`):

```bash
python3 -m http.server 8000
```

Depois abra no navegador:

- **Mínima** — http://localhost:8000/ondaesfera.html
- **Completa** — http://localhost:8000/ondanaesfera-glm.html

## Aplicativos

### `ondaesfera.html` — versão mínima
Esfera com auto-rotação, clique para excitar e visualização por cores nos vértices. Cerca de 140 linhas, sem controles externos (Three.js r0.158).

### `ondanaesfera-glm.html` — versão completa
Three.js r0.128 + `OrbitControls`, `ShaderMaterial` personalizado, painel de controles (velocidade, amortecimento, força do impacto, volume), botão de reiniciar, legenda de cores e **áudio por modelagem física** do tambor.

Controles:

- **Clique** — toca o tambor (cria um impulso gaussiano no ponto de impacto)
- **Arraste** — rotaciona a câmera
- **Rolar** — zoom

## Como funciona

### Física da onda
Método de diferenças finitas para a equação da onda sobre a malha da esfera, com um laplaciano discreto calculado sobre os vizinhos de cada vértice. Os buffers `u`, `uPrev`, `uNext` (`Float32Array`) são trocados a cada frame. A vizinhança é pré-computada uma única vez (as posições da CPU não mudam — o deslocamento acontece apenas no *vertex shader*), e vértices duplicados na costura/polos são tratados para a onda atravessar continuamente.

### Áudio (apenas na versão completa)
O som **não** é um efeito disparado na batida: é a própria membrana vibrando. Um `AudioWorklet` integra, na taxa de áudio (~44,1 kHz), as equações dos modos normais da esfera — cada modo é um oscilador harmônico amortecido

```
q'' + 2γ q' + ω_l² q = 0,  com  ω_l = ω₁·√(l(l+1)/2)
```

ou seja, as autofrequências do operador de Laplace–Beltrami na esfera (parciais **inarmônicos** 1 : 1,73 : 2,45 : ..., o timbre característico de membrana). Melhorias de timbre sobre o modelo modal puro:

- **Degenerescência modal** — cada grau `l` vira sub-osciladores levemente desafinados, gerando batimentos e *shimmer*.
- **Dureza da batida** — a força molda o espectro (golpe forte = mais agudos), não só o volume.
- **Glide de tensão** — amplitude alta sobe o tom e relaxa ao decair (o *doom* de tímpano/tom).
- **Estágio de saída** — bloqueio de DC + saturação suave (`tanh`) para corpo e calor.

Mapeamento dos controles: velocidade → tom (ω₁), amortecimento → decaimento (γ), força do impacto → energia da batida.

## Tecnologias

- JavaScript puro (sem etapa de build)
- Three.js (via CDN)
- Diferenças finitas para a onda; Web Audio (`AudioWorklet`) para o som
- Interface em português (pt-BR)

## Licença

Sem licença definida. Adicione uma se pretende permitir reuso.
