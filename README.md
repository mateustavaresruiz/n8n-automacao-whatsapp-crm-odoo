# n8n-automacao-whatsapp-crm-odoo
Automação feita no n8n que salva informações personalizadas da conversa do whatsapp como nome completo, endereço, número e email no CRM da ferramenta Odoo. O legal que pode ser qualquer tipo de informação que esteja na conversa pode ser salvo apenas ajustar no prompt da IA e na coluna do Sheets.

1. 🔗 Webhook
Recebe os dados do WhatsApp via requisição POST.

Cada usuário é identificado pelo número de telefone (ID).

2. 🧹 Filtra Mensagens Recebidas
Apenas mensagens relevantes (texto) são processadas.

Exclui áudios, imagens e outros tipos de mídia.

3. 🤖 GPT 4.1-nano (IA)
A IA analisa o conteúdo textual da mensagem.

O prompt foi treinado para identificar:

Nome completo, Endereço, Telefone, E-mail

Se a mensagem contiver uma dessas informações, ela é transformada em um JSON.

4. 🧪 Formata Número WhatsApp + Separador JSON
Garante que o número do usuário esteja formatado corretamente.

Divide e organiza o JSON para os próximos passos.

5. 📄 Cria Linha / Coleta Linha (Google Sheets)
Cria Linha: se o número do WhatsApp (ID) ainda não estiver presente, cria uma nova linha no Google Sheets.

Coleta Linha: busca os dados existentes para este ID e prepara para fusão.

6. 🔄 Unifica Sheets + IA
Junta os dados antigos do usuário com as novas informações coletadas pela IA.

Garante que apenas campos novos sejam adicionados e que nada existente seja sobrescrito.

7. 🧱 Unifica JSON
Reestrutura os dados combinados em um JSON unificado, compatível com o formato do Sheets.

8. ✍️ Insere os Valores (Google Sheets)
Atualiza a linha do usuário com as informações novas.

Evita duplicações e garante consistência no banco de dados.

9. 🔍 Verifica os Campos no Sheets
Confere se todos os campos essenciais (nome, e-mail etc.) estão preenchidos.

Essa etapa previne o envio incompleto para o CRM.

10. 🧾 Cria Contato no CRM (Odoo)
Se todos os dados necessários estiverem presentes, cria (ou atualiza) o contato no CRM.

A ação no Odoo é: create: opportunity.

(Infelizmente não consigo compartilhar o projeto pois estou usando o plano gratuito. Com os códigos fornecidos fica muito fácil de copiar)
![image](https://github.com/user-attachments/assets/cd9f427d-6931-424f-8fd1-553a2679e644)

Informações e códigos importantes: 

Prompt do agente de IA:
Analise a seguinte mensagem de WhatsApp e diga se ela contém alguma das seguintes informações: nome completo, email, telefone, endereço. Responda em JSON com as chaves exatamente assim: "nome", "email", "endereco", "telefone".
O endereço pode ser região como apenas o estado por exemplo ou lista de cidades, dependendo do que cada usuário mandar, pode ser variado e diversos sentidos, fique atento a isso.
Se não contiver uma informação, retorne null para o campo.
Mensagem: 
telefone: 
{{ $json.body.data.message.conversation }}

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Formatar número de WhatsApp:
const raw = $('Webhook').first().json.body.data.key.remoteJid;
const phone = raw.replace('@s.whatsapp.net', '');
return [{ json: { phone } }];

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Unifica os JSON:

const [entrada1, entrada2] = items;

function preencherSeVazio(orig, novo) {
  return (!orig || orig === "") && novo ? novo : orig;
}

return [
  {
    json: {
      número_da_linha: entrada1.json["número_da_linha"],
      Nome: preencherSeVazio(entrada1.json["Nome"], entrada2.json["Nome"]),
      Email: preencherSeVazio(entrada1.json["Email"], entrada2.json["Email"]),
      Endereco: preencherSeVazio(entrada1.json["Endereco"], entrada2.json["Endereco"]),
      Telefone: preencherSeVazio(entrada1.json["Telefone"], entrada2.json["Telefone"])
    }
  }
];

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Verifica se os campos no sheets:
const item = items[0].json;

const camposObrigatorios = ['Telefone', 'Nome', 'Email', 'Endereco'];
const linhaCompleta = camposObrigatorios.every(campo => item[campo] && item[campo].toString().trim() !== "");

if (linhaCompleta) {
  return [ { json: item } ]; // Continua para o próximo nó
} else {
  return []; // Impede envio ao Odoo
}

