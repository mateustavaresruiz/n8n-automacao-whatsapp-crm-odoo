# Automação WhatsApp + Google Sheets + CRM Odoo (n8n)

Este fluxo do **n8n** automatiza o registro de informações recebidas por mensagens de WhatsApp, interpretando os dados com ajuda de IA e salvando-os de forma organizada no **Google Sheets**, além de criar contatos automaticamente no **CRM Odoo**.
O mais legal é que você pode personalizar para a IA filtrar qualquer tipo de mensagem e trabalhar de que forma quiser basta adaptar, no meu caso usei para adicionar contatos no Odoo. (Infelizmente não consigo compartilhar o projeto pois estou usando o plano gratuito. Com os códigos fornecidos fica muito fácil de copiar)


Integrações feitas: 
Evolution API: Para receber mensagens do Whatsapp diretamente no meu fluxo atráves de um Webhhok.
Google Outh2: Para conectar o google drive com o n8n.
Odoo API: Para criar contatos no CRM.


## Visão Geral do Fluxo

1. Recebimento de mensagens do WhatsApp por meio de um webhook.
2. Filtragem de mensagens de texto.
3. Uso de IA para identificar dados úteis: nome, telefone, e-mail e endereço.
4. Formatação do número de WhatsApp para uso como identificador.
5. Verificação no Google Sheets se o contato já existe.
6. Criação ou atualização da linha no Sheets com os dados extraídos.
7. Unificação dos dados antigos com os novos.
8. Verificação de campos obrigatórios antes do envio ao CRM.
9. Criação de contato no CRM Odoo.

---

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

