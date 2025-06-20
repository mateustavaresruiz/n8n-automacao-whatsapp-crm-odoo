# n8n-automacao-whatsapp-crm-odoo
Automação feita no n8n que salva informações da conversa do whatsapp como nome completo, endereço, número e email no CRM da ferramenta Odoo

(Infelizmente não consigo compartilhar o projeto pois estou usando o plano gratuito.)
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

