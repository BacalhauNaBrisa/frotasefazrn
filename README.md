<h1 align="center">frotasefazrn</h1>

<p align="center">
  Sistema não-oficial de agendamento e acompanhamento em tempo real da<br/>
  <strong>frota de veículos da SUMAT</strong> (Subcoordenadoria de Fiscalização de Mercadorias em Trânsito) da Secretaria da Fazenda do Rio Grande do Norte (Sefaz/RN).
</p>

<p align="center">
  🔗 <strong><a href="https://bacalhaunabrisa.github.io/frotasefazrn/">bacalhaunabrisa.github.io/frotasefazrn</a></strong>
</p>

---

## Sobre o projeto

Este repositório contém uma única página HTML autocontida (`index.html`) que permite:

1. **Agendar** a retirada e a devolução de uma das 56 viaturas da frota, verificando em tempo real quais estão disponíveis num determinado local, período e finalidade;
2. **Acompanhar**, ao vivo, o status de toda a frota (em uso / disponível), com filtros e ordenação por coluna.

Não há servidor, backend ou build: basta abrir o arquivo no navegador (ou acessá-lo via GitHub Pages). O projeto é uma ferramenta de apoio operacional interno à SUMAT, sem qualquer vínculo institucional oficial com a Sefaz/RN.

## Como usar

### 1. Agendamento

1. Informe a data e a hora de retirada e a data e a hora de devolução (fuso de Recife, UTC−3). É permitido retirar e devolver no mesmo dia.
2. Digite a matrícula (7 dígitos) do servidor responsável; ela é validada automaticamente contra a listagem cadastrada e formatada como `256.410-6`.
3. Selecione o local de retirada, o setor solicitante e a finalidade (Itinerância, Plantão ou Visita Institucional).
4. Clique em **"Verificar disponibilidade"**. O botão só é habilitado quando todos os campos estão preenchidos e a matrícula é válida.
5. O sistema valida se a retirada está no futuro em relação ao horário atual de Recife e lista as viaturas do local escolhido que não têm nenhuma reserva com horário sobreposto ao período solicitado.
6. Clique no ícone verde de confirmação (✔) na linha da viatura desejada para gravar a reserva. Uma última verificação é feita nesse momento, para o caso de outra pessoa ter reservado a mesma viatura ou o horário já não estar mais no futuro.

### 2. Frota de veículos

Logo abaixo, a tabela "Frota de veículos" mostra o status ao vivo das 56 viaturas — recalculado automaticamente a cada 15 segundos e a cada nova reserva —, incluindo veículo, placa, caracterização, local de lotação, propriedade, status (**EM USO**, em vermelho, ou **DISPONÍVEL**, em verde), responsável, setor solicitante, finalidade e datas/horas de retirada e devolução. Clique no título de qualquer coluna para ordenar (crescente/decrescente) e use os campos logo abaixo dos títulos para filtrar por texto ou por valor.

## Lógica de disponibilidade

Todo o cálculo roda **no navegador do usuário**, em JavaScript puro (nenhum dado é processado em um servidor próprio).

- Uma viatura é candidata se estiver lotada no local selecionado.
- Uma viatura candidata é considerada **disponível** para o período solicitado se nenhuma reserva já registrada para ela tiver sobreposição de horário com o período pedido (retirada₁ < devolução₂ **e** retirada₂ < devolução₁).
- No dashboard "Frota de veículos", o status **EM USO** é atribuído, em tempo real, à viatura que tiver uma reserva cujo intervalo [retirada, devolução) contenha o instante atual (horário de Recife); caso contrário, o status é **DISPONÍVEL**.
- Todas as datas/horas informadas nos formulários são interpretadas como horário de Recife (UTC−3 fixo, já que o Brasil não adota mais horário de verão), independentemente do fuso horário do dispositivo de quem estiver usando a página.

### Persistência

As reservas ficam guardadas em um banco de dados na nuvem (Firebase Realtime Database) e são sincronizadas em tempo real entre todos os usuários que acessarem a página — ver seção [Sincronização entre todos os usuários](#sincronização-entre-todos-os-usuários) abaixo. Enquanto o Firebase não estiver configurado, o sistema funciona em **modo local**: cada reserva fica salva apenas no `localStorage` do navegador de quem a fez, sem aparecer para as demais pessoas, e um aviso amarelo é exibido no topo do dashboard de agendamento avisando disso.

## Sincronização entre todos os usuários

Por padrão, uma página hospedada no GitHub Pages é **estática**: não existe servidor nem banco de dados próprios, então, sem nenhuma configuração adicional, cada navegador só enxergaria as próprias reservas. Para que todos os servidores vejam e registrem as **mesmas** reservas, o `index.html` se conecta a um banco de dados gratuito do Google — o [Firebase Realtime Database](https://firebase.google.com/docs/database) — diretamente do navegador, via JavaScript, sem precisar de nenhum servidor mantido por vocês.

Essa configuração precisa ser feita uma única vez, por qualquer pessoa com uma conta Google:

1. Acesse [console.firebase.google.com](https://console.firebase.google.com/) e faça login com uma conta Google.
2. Clique em **"Adicionar projeto"**, dê um nome (por exemplo, `frotasefazrn`) e conclua a criação. Não é necessário ativar o Google Analytics.
3. Dentro do projeto, no menu lateral, acesse **Build → Realtime Database** e clique em **"Criar banco de dados"**.
4. Escolha uma localização (qualquer uma serve) e, quando perguntado sobre as regras de segurança, escolha iniciar em **modo de teste** — ou já configure manualmente as regras do passo 5.
5. Na aba **"Regras"** do Realtime Database, substitua o conteúdo pelo seguinte e publique:
   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```
   ⚠️ **Nota de segurança:** essas regras deixam o banco de dados com leitura e escrita **públicas** (sem exigir login), pois o sistema não usa autenticação — qualquer pessoa com o link do projeto poderia, em tese, alterar os dados diretamente pela API do Firebase. Isso é adequado para este uso informal e interno entre os servidores da SUMAT, mas o banco **não deve ser reaproveitado** para guardar informações sensíveis.
6. Vá em **⚙️ Configurações do projeto → Geral**, role até **"Seus apps"** e clique no ícone `</>` (Web) para registrar um novo app. Dê um apelido qualquer (ex.: `frotasefazrn-web`) e clique em **"Registrar app"** (não é necessário adicionar o Firebase Hosting).
7. Copie o objeto `firebaseConfig` exibido na tela — algo como:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "frotasefazrn.firebaseapp.com",
     databaseURL: "https://frotasefazrn-default-rtdb.firebaseio.com",
     projectId: "frotasefazrn",
     storageBucket: "frotasefazrn.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:abcdef1234567890abcdef"
   };
   ```
8. Abra o arquivo `index.html` deste repositório e localize o bloco `var FIREBASE_CONFIG = { ... }`, no início do `<script>` final da página. Substitua os valores de exemplo pelos valores copiados no passo 7.
9. Salve, faça o commit e o push do `index.html` atualizado para o repositório — o GitHub Pages publica a nova versão automaticamente em alguns minutos.

A partir daí, a barra de status acima do formulário de agendamento passa a exibir **"Sincronizado com todos os usuários"** (em vez de "Modo local"), e qualquer reserva feita por alguém aparece, em tempo real, para todas as outras pessoas com a página aberta.

## Dados cadastrados

- **8 locais**: Sede Sefaz, NIF, 1 URT, 2 URT, 3 URT, 5 URT, 6 URT e 7 URT.
- **19 setores solicitantes**: SUMAT, SUMAT/NIF, SUCADI, SUFISE, SUSCOMEX, COFIS, CACE, SUDEFI, GS, Central de Veículos, COGEF, COTIC, 1 URT, 2 URT, 3 URT, 5 URT, 6 URT, 7 URT e COEF.
- **3 finalidades**: Itinerância, Plantão e Visita Institucional.
- **48 servidores** cadastrados por nome e matrícula, usados para validar o campo de matrícula no agendamento.
- **56 viaturas** da frota, com veículo, placa, caracterização, local de lotação e propriedade (própria/alugada).

Qualquer alteração nessas listas (troca de servidor, movimentação de viatura entre locais etc.) exige editar diretamente os arrays correspondentes (`LOCAIS`, `SETORES`, `FINALIDADES`, `SERVIDORES`, `VEICULOS`) no início do `<script>` do `index.html`.

## Tecnologias utilizadas

- **HTML5** — estrutura da página, em arquivo único (`index.html`).
- **CSS3** puro — sem framework; variáveis CSS (`:root`) para cores/tema, tabelas com colunas fixas (`position: sticky`) para o cabeçalho e a linha de filtros do dashboard de frota, e fontes do Google Fonts (*Space Grotesk*, *Inter*, *JetBrains Mono* para números e placas).
- **JavaScript (ES5/ES6)** — geração dos formulários e da tabela de frota, validação de matrícula, cálculo de disponibilidade por sobreposição de horário e ordenação/filtro das colunas.
- **jQuery 3.7** (via CDN) — manipulação do DOM e eventos que disparam a atualização automática.
- **Firebase Realtime Database** (via CDN, SDK compat) — sincronização em tempo real das reservas entre todos os usuários que acessam a página; com `localStorage` como reserva local enquanto o Firebase não estiver configurado.
- **GitHub Pages** — hospedagem estática, sem backend próprio, sem build step, sem dependências instaladas: o repositório é publicado como está.

Não há framework de front-end (React, Vue etc.), bundler ou etapa de compilação — o projeto é intencionalmente simples para poder ser mantido e publicado direto pela interface do GitHub, do mesmo modo que os projetos irmãos [`lotacaosefazrn`](https://github.com/BacalhauNaBrisa/lotacaosefazrn), [`remunerasefazrn`](https://github.com/BacalhauNaBrisa/remunerasefazrn) e [`cebraspe`](https://github.com/BacalhauNaBrisa/cebraspe).

## Estrutura do repositório

```
frotasefazrn/
├── index.html   # página única: HTML + CSS + JS embutidos
└── README.md    # este arquivo
```

## Aviso legal

Sistema independente e não-oficial, sem qualquer vínculo institucional com a Sefaz/RN. Os dados de locais, setores, servidores e viaturas aqui reproduzidos têm caráter operacional interno à SUMAT e podem ficar desatualizados caso a composição real da frota ou do quadro de servidores mude. Consulte sempre a chefia da SUMAT para fins oficiais antes de tomar qualquer decisão com base nos resultados desta página.
