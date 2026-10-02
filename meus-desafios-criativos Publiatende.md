Atue como um especialista em N8N.

Crie uma automação para triagem de contatos no WhatsApp, agendamento de reuniões para leads e notificação da equipe.

Público: Equipe de atendimento, equipe comercial e clientes/leads da agência de publicidade.

Ferramentas envolvidas: WhatsApp Business (API), Google Sheets, Google Calendar e Gmail.

Fluxo:

1. Receber mensagem do WhatsApp via Webhook.  
2. Normalizar o número de telefone e consultar na planilha de clientes ativos (Google Sheets).  
3. Triar o contato via condicional (cliente ativo vs. novo lead).  
4. Se for cliente ativo: enviar resposta automática de direcionamento no WhatsApp e alertar a equipe via Gmail.  
5. Se for novo lead: responder com mensagem de boas-vindas e link para agendamento de reunião diagnóstica.  
6. Receber confirmação do agendamento (via Webhook/Google Calendar), registrar o evento e notificar o grupo de atendimento por e-mail com os detalhes do lead e link do Meet.

Regras:

* Normalizar números de telefone (remover caracteres especiais, validar DDI, DDD e variação do 9º dígito) antes da consulta.  
* Ignorar mensagens automáticas, atualizações de status/stories ou mensagens enviadas pelo próprio bot para evitar loops.  
* Prever fallback com opção de texto ("Já sou cliente") para clientes que usem números novos não cadastrados.  
* Estruturar em dois Webhooks/workflows independentes: um para a conversa/triagem inicial e outro para a captura e notificação do agendamento.

### 

### 

### 

### 

Explique quais nós do N8N devem ser utilizados e a lógica de funcionamento do workflow.

### **Nós do N8N e Lógica de Funcionamento**

A solução é estruturada em dois fluxos (ou dois pontos de entrada independentes) para evitar dependência síncrona entre a conversa do chat e a confirmação da agenda.

#### **Fluxo 1: Triagem e Roteamento de Contatos (WhatsApp)**

1. **Webhook (`Webhook`)**  
   * **Função:** Recebe o payload HTTP (método POST) disparado pela API do WhatsApp sempre que uma nova mensagem chega.  
2. **Filtro Anti-Loop (`Filter` ou `If`)**  
   * **Função:** Avalia se o remetente é o próprio bot (`fromMe: true`) ou se a mensagem é evento de sistema/status. Se for, a execução é encerrada imediatamente.  
3. **Normalização do Número (`Code`)**  
   * **Função:** Executa um script em JavaScript para limpar o número (remover `@s.whatsapp.net`, hífens e parênteses) e padronizar o padrão brasileiro (55 \+ DDD \+ 8 ou 9 dígitos), garantindo correspondência exata na base.  
4. **Consulta de Base (`Google Sheets`)**  
   * **Operação:** `Get Many Rows` (ou `Search`).  
   * **Função:** Busca o número normalizado na coluna de telefones da planilha de clientes ativos da agência.  
5. **Decisão de Triagem (`If`)**  
   * **Condição:** Verifica se a consulta anterior retornou registros (`{{ $json.length > 0 }}`).  
   * **Branch True (Cliente Ativo):**  
     * **`HTTP Request`:** Dispara mensagem via API do WhatsApp informando opções de atendimento operacional/financeiro.  
     * **`Gmail`:** Dispara um e-mail para a equipe interna informando que um cliente ativo iniciou contato via WhatsApp.  
   * **Branch False (Novo Lead):**  
     * **`HTTP Request`:** Envia mensagem de boas-vindas com o link de agendamento (Google Calendar Appointment Schedule ou ferramenta integrada) e a opção de digitar "Já sou cliente" caso o número seja recente.

#### **Fluxo 2: Captura do Agendamento e Notificação da Equipe**

1. **Webhook de Agendamento (`Webhook` ou `Google Calendar Trigger`)**  
   * **Função:** Disparado no momento em que o lead conclui a escolha de data e horário na página de agendamento.  
2. **Atualização / Registro (`Google Sheets`)**  
   * **Operação:** `Append Row`.  
   * **Função:** Grava os dados do lead (nome, e-mail, telefone, empresa e data/hora marcada) em uma aba de "Leads Agendados".  
3. **Notificação por E-mail (`Gmail`)**  
   * **Operação:** `Send Message`.  
   * **Destinatário:** Grupo/e-mail de atendimento da equipe comercial.  
   * **Conteúdo:** Mensagem estruturada em HTML contendo nome do lead, contato, objetivo da reunião, horário e link gerado do Google Meet para acesso da equipe

