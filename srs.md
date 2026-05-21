# SRS — Software Requirements Specification
## Simulador de Estresse e Conforto Térmico Habitacional

**Projeto Interdisciplinar — 2026**
Michelle Di Loraine Brito Peixoto (RU 5614521) · Cleyton Duda Macedo (RU 5704345)

---

## 1. Introdução

### 1.1 Propósito
Este documento especifica os requisitos do **Simulador de Estresse Térmico**, uma aplicação web interativa que modela o comportamento térmico de uma unidade habitacional ao longo de um ciclo de 24 horas, permitindo experimentação virtual com parâmetros construtivos e sistemas de resfriamento.

### 1.2 Escopo
O simulador atua como **complemento digital** ao Kit de Experimentos Físicos de Baixo Custo, ampliando o alcance pedagógico do projeto. Enquanto o kit físico permite observação direta da absorção térmica em folhas de papel de diferentes cores, o simulador escala a experimentação para unidades habitacionais completas, incorporando variáveis que seriam impossíveis de reproduzir em laboratório escolar (sol em arco real, ciclo dia-noite, sistemas de climatização).

### 1.3 ODS Alinhadas
- **ODS 4 — Educação de Qualidade** (principal): democratização do acesso a ferramentas experimentais de física
- **ODS 11 — Cidades e Comunidades Sustentáveis** (relacionada): conscientização sobre habitabilidade térmica e eficiência energética em construções de baixo custo

### 1.4 Tecnologia
Single-file HTML5 com Canvas 2D nativo, sem dependências externas além de Google Fonts. Executa offline em qualquer navegador moderno. Zero instalação.

---

## 2. Visão Geral do Sistema

### 2.1 Objetivo Funcional
Permitir ao usuário responder, de forma quantitativa e visual, perguntas como:
- Quanto a cor das paredes impacta o ganho térmico de uma casa?
- Em que momento do dia o calor acumulado atinge o pico?
- Um ar-condicionado de 12.000 BTU é suficiente para uma casa de 80 m²?
- Por que um climatizador evaporativo falha em regiões úmidas?
- Quanto tempo o ambiente leva para resfriar naturalmente após o pôr do sol?

### 2.2 Princípios Físicos Implementados
- **Curva senoidal de temperatura externa** (pico às 14h, mínima às 5h)
- **Arco solar real** (nasce 6h, zênite 12h, põe 18h) com irradiância proporcional
- **Absorção solar** calculada por luminância percebida do espectro CMYK (Albedo)
- **Condução bidirecional** através do envelope (lei de Fourier)
- **Infiltração de ar** por taxa de trocas horárias (ACH)
- **Radiação noturna** (Stefan-Boltzmann, apenas do telhado)
- **Resfriamento evaporativo** governado por temperatura de bulbo úmido (Stull)
- **Heat Index** (Rothfusz/NWS) para sensação térmica
- **Convecção forçada** (ventilador via lei de Newton)
- **2º princípio da termodinâmica** rigorosamente respeitado

---

## 3. Requisitos Funcionais

### 3.1 Painel Esquerdo — Parâmetros de Entrada

#### 3.1.1 Seção Estrutura
| Controle | Função | Faixa |
|---|---|---|
| **Área** | Define a área construída da unidade habitacional em m². Afeta diretamente o ganho solar, a massa térmica e a área de envelope (paredes + teto). | 10 – 500 m² |
| **Pé-direito** | Altura interna do ambiente em metros. Determina o volume de ar a ser climatizado e a área das paredes. | 2 – 6 m |

#### 3.1.2 Seção Ambiente Externo
| Controle | Função | Faixa |
|---|---|---|
| **Pico T°** | Temperatura externa máxima do dia (atingida às 14h). Define o estresse térmico máximo a que a unidade será submetida. | 20 – 55 °C |
| **T° mínima** | Temperatura externa mínima do dia (atingida às 5h). Estabelece o piso da curva noturna. | 10 – 35 °C |
| **Umidade** | Umidade relativa do ar ambiente em %. **Crítico** para o desempenho do climatizador evaporativo. | 10 – 100 % |

#### 3.1.3 Seção Cor das Paredes (CMYK)
Quatro sliders no padrão de impressão CMYK que controlam visualmente a cor das paredes da casa renderizada e, fisicamente, o coeficiente de absorção solar (albedo).

| Slider | Função |
|---|---|
| **C — Cian** | Componente ciano. Reduz reflexão na faixa do vermelho. |
| **M — Magenta** | Componente magenta. Reduz reflexão na faixa do verde. |
| **Y — Amarelo** | Componente amarelo. Reduz reflexão na faixa do azul. |
| **K — Preto** | Componente preto puro. Reduz luminância global e aumenta absorção térmica de forma quase linear. |

**Indicadores em tempo real:**
- Amostra visual da cor (swatch)
- Código hexadecimal RGB
- Barra de Absorção Solar (10 % branca → 95 % preta)
- **Watts absorvidos no momento atual** — mostra impacto físico instantâneo

#### 3.1.4 Seção Sistemas de Resfriamento
Três blocos expansíveis com toggle on/off e parâmetro de potência.

| Sistema | Controle | Função Física |
|---|---|---|
| **❄ Ar Condicionado** | Potência em BTU (5.000 – 80.000) | Reduz T° real **e** umidade. Único capaz de derrubar T° interna abaixo da externa. |
| **💧 Climatizador Evap.** | Vazão em m³/h (50 – 5.000) | Reduz T° real, **eleva** umidade. Eficiência governada por bulbo úmido — inútil em ar saturado (>85 % UR). Mostra eficiência em tempo real. |
| **🌀 Ventilador** | Velocidade 1–5 | **Não altera T° real**. Reduz apenas sensação térmica via convecção forçada. Ineficaz quando T° > 34 °C (temperatura da pele). |

---

### 3.2 Área Central — Canvas de Simulação

#### 3.2.1 Renderização Visual
Cena completa desenhada em Canvas 2D em tempo real (60 fps):
- **Céu** com gradiente em 8 fases (madrugada → amanhecer → manhã → meio-dia → tarde → entardecer → crepúsculo → noite)
- **Sol** percorrendo arco parabólico real do horizonte leste ao oeste
- **Lua** com fase crescente percorrendo arco noturno
- **Estrelas** com cintilação durante período noturno
- **Nuvens** deslizando horizontalmente durante o dia
- **Montanhas** em duas camadas com tonalidade adaptada ao período do dia
- **Árvores** (coníferas) flanqueando a cena
- **Casa** com paredes coloridas pelo CMYK, telhado, porta, janelas que acendem à noite
- **Sombra projetada** dinâmica, com direção oposta ao sol, comprimento variável com altura solar
- **Shimmer térmico** acima do telhado quando T° interna ultrapassa limites de conforto

#### 3.2.2 Overlay Superior Esquerdo — Card de Status
Card semi-transparente com:
- Temperatura externa em LED colorido (azul/verde/âmbar/vermelho)
- Sensação térmica
- Relógio analógico miniatura + readout digital
- Indicador de período (Amanhecer/Manhã/Tarde/Entardecer/Noite)

#### 3.2.3 Overlay Inferior Esquerdo — Card de Alerta
Card vermelho com fundo preto que aparece somente quando a capacidade de resfriamento dos equipamentos ativos é insuficiente para vencer a carga térmica. Exibe déficit em watts e percentual.

---

### 3.3 Barra Inferior — Controles de Tempo

| Controle | Função |
|---|---|
| **Display Digital (HH:MM)** | Mostra a hora atual da simulação. |
| **Timeline de 24h** | Slider arrastável que permite "scrubar" qualquer momento do dia. Marcações em 00h, 06h, 12h, 18h e 24h. |
| **▶ / ⏸ Play/Pause** | Inicia ou pausa o avanço do tempo. |
| **Slider Aceleração** | Curva exponencial 1× → 600×. Verde (tempo aproximado real), índigo (rápido), âmbar (muito rápido), vermelho (warp). Marcações em 1×, 8×, 60×, 220× e 600×. |
| **Readout de Velocidade** | Exibe multiplicador atual em cor correspondente. |
| **↺ Reset Velocidade** | Volta aceleração instantaneamente a 1×. |

---

### 3.4 Header — Controles Globais

| Controle | Função |
|---|---|
| **Logo "Projeto Interdisciplinar"** | Identificação do projeto. |
| **↺ RESET** | Restaura todos os parâmetros aos valores padrão, pausa a simulação, volta tempo a 06:00 (amanhecer) e zera CMYK e equipamentos. |
| **Badge ⚡ Velocidade** | Mostra aceleração atual. Muda para âmbar/vermelho em altas velocidades. |
| **Badge Status** | "● RODANDO" (verde) ou "● PAUSADO" (cinza). |

---

### 3.5 Painel Direito — Dashboard

Painel de monitoramento em tempo real, atualizado a cada frame.

| Métrica | Significado |
|---|---|
| **Temperatura Externa** | Valor instantâneo da curva senoidal. Mostra período do dia e fator solar atual (%). |
| **Temperatura Interna Real** | Resultado da integração térmica. Barra colorida indica conforto (verde < 24 < âmbar < 30 < vermelho). |
| **Sensação Térmica** | Heat Index ajustado por efeito ventilador. Sub-label classifica conforto. |
| **Umidade Interna** | RH% resultante dos efeitos de equipamento. Barra colorida (ciano < âmbar < vermelho). |
| **Balanço Térmico Líquido** | Soma algébrica de **todos os fluxos** (solar + condução + infiltração + radiação). Positivo (vermelho) = casa esquentando. Negativo (ciano) = casa esfriando. Sub-label decompõe ☀solar · 🏠condução · 💨infiltração · 🌌radiação. |
| **Capacidade de Resfriamento** | Potência total dos equipamentos ativos. Barra colorida indica suficiência (verde ≥ 100%, âmbar ≥ 60%, vermelho < 60%). |
| **Tempo para Atingir 24°C** | Estimativa baseada em balanço energético. "∞" se sistema for incapaz. |
| **Curva de Temperatura (24h)** | Gráfico mini com cursor de tempo, mostrando ciclo completo previsto. |

---

## 4. Cenários de Uso Pedagógico

### 4.1 Demonstração: Impacto da Cor das Paredes
1. Reset → avançar tempo para 13:00 (meio-dia)
2. Observar Balanço Térmico inicial (~+800W com paredes brancas)
3. Mover slider K para 100%
4. Observar salto instantâneo para ~+7600W
5. Acelerar para 60×, observar curva de aquecimento muito mais agressiva

### 4.2 Demonstração: Limitação do Climatizador em Ar Úmido
1. Definir Umidade = 90%
2. Pico T° = 36°C, avançar para 14:00
3. Ativar Climatizador, observar eficiência ≈ 15%
4. Repetir com Umidade = 30% → eficiência ≈ 80%

### 4.3 Demonstração: Inércia Térmica e Resfriamento Noturno
1. Reset, executar dia inteiro em 60× sem equipamentos
2. Observar casa atingir pico ~40°C às 16h
3. Continuar simulação durante noite
4. Observar resfriamento gradual **limitado pela temperatura externa** (nunca abaixo)

### 4.4 Demonstração: Dimensionamento de Ar-Condicionado
1. Casa 80 m², Pico 38°C, paredes pretas (K=100%)
2. Ativar AC com 7.000 BTU → alerta de ineficiência
3. Aumentar para 24.000 BTU → tempo de resfriamento finito
4. Discutir relação BTU/m² em projetos residenciais

---

## 5. Requisitos Não Funcionais

| Categoria | Requisito |
|---|---|
| **Compatibilidade** | Chrome, Firefox, Edge, Safari (versões recentes) |
| **Performance** | Renderização contínua a 60 fps em hardware modesto |
| **Acessibilidade** | Layout responsivo até 768 px de largura |
| **Distribuição** | Arquivo único HTML executável offline, sem dependências locais |
| **Idioma** | Português brasileiro (pt-BR) em toda a interface |
| **Persistência** | Stateless por sessão (recarregar resseta tudo) |

---

## 6. Restrições e Premissas

- Modelo unidimensional simplificado (não considera orientação cardinal específica)
- Temperatura tratada como uniforme em todo o volume interno (sem estratificação)
- Material das paredes assumido como alvenaria simples (U = 1.8 W/m²K)
- Latitude/sazonalidade não parametrizadas (curva solar genérica)
- O simulador é uma ferramenta **didática**, não substitui projetos térmicos de engenharia profissional