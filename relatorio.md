# Joao Ramos DR4 TP3
> Por João Victor Cicero de M. T. Ramos.
> *Novembro de 2026*

---

# Exercício 1

### Contexto

A Vetor Security avança o engajamento contra o crAPI para a fase de exploração de autorização. 

O primeiro alvo é a Broken Object Level Authorization (BOLA), a categoria historicamente mais explorada em incidentes reais de APIs REST e responsável por diversos vazamentos de dados públicos envolvendo aplicações de veículos conectados e serviços financeiros. A falha ocorre quando a API verifica se o usuário está autenticado, mas não verifica se ele é o dono do recurso solicitado. 

Você recebeu dois pares de credenciais válidas de usuários distintos no ambiente de laboratório para conduzir um teste comparativo controlado, sem nunca comprometer contas fora do escopo autorizado.

### Tarefa
1. Autentique-se como Usuário A, capture uma requisição que acesse um recurso pertencente a ele (por exemplo, um veículo cadastrado) e identifique o identificador do recurso na requisição.

2. Realize o teste A-B: substitua o identificador do recurso pelo de um recurso pertencente ao Usuário B, mantendo a sessão do Usuário A, e registre a resposta obtida.

3. Conclua se a API está vulnerável a BOLA nesse endpoint e documente o impacto de negócio da falha encontrada.

### Item 1.1
Podemos verificar o recurso criado pelo usuário A e podemos verificar que foi criado por ele através do código *VIM* do veiculo.

![/Exercicio_1/imagem.1.1.1.png](/Exercicio_1/imagem.1.1.1.png)

![/Exercicio_1/imagem.1.1.2.png](/Exercicio_1/imagem.1.1.2.png)

Avaliando a requisição capturada, as informações na tela e o url no navegador, podemos verificar que o ID do objeto `service_report` é `6`

![/Exercicio_1/imagem.1.1.3.png](/Exercicio_1/imagem.1.1.3.png)

![/Exercicio_1/imagem.1.1.4.png](/Exercicio_1/imagem.1.1.4.png)

### Item 1.2
Evidencia do usuário A abrindo um `service_report` criado pelo usuario B. podemos verificar que o veiculo relacionado *Não pertence* ao usuario A através do código *VIM* do veículo

![/Exercicio_1/imagem.1.2.1.png](/Exercicio_1/imagem.1.2.1.png)

### Item 1.3

As evidencias mostram que a API está vulneravel a BOLA, quberando com o principio de confidencialidade, já que um usuario consegue acessar as informações de outro. 

---

# Exercício 2

### Contexto

Além da autorização a nível de objeto, a Vetor Security pede o teste de Broken Function Level Authorization (BFLA): verificar se um usuário sem privilégios administrativos consegue acessar funções restritas a administradores, como gestão de outros usuários ou configurações internas do sistema. 

Esse tipo de falha costuma passar despercebido em revisões de código porque a checagem de papel muitas vezes existe apenas na interface visual da aplicação (o botão administrativo simplesmente não aparece para o usuário comum), mas não é reforçada no backend, que aceita a requisição de qualquer usuário autenticado independentemente do papel atribuído. 

Times de desenvolvimento sob pressão de prazo tendem a confiar demais na camada visual como controle de acesso, um erro que este exercício deve expor de forma concreta e reproduzível.

### Tarefa

1. Identifique, no mapa de endpoints já levantado, funções que aparentam ser restritas a administradores (por exemplo, gestão de usuários ou configurações do sistema).
2. Utilizando um token de usuário comum, tente acessar diretamente essas funções administrativas e registre o resultado.
3. Justifique tecnicamente por que a checagem de autorização precisa ocorrer no backend e não apenas na camada de apresentação, com base no resultado observado.

### Item 2.1
Utilizando a listagem de endpoints do postman criada no TP2, podemos identificar os seguintes endpoints que aparentam ser restritos a administradores:
- `DELETE /identity/api/v2/admin/videos/{video_id}`
- `POST /community/api/v2/coupon/new-coupon`
- `GET /workshop/api/management/users/all`
- `POST /workshop/api/shop/products`


### Item 2.2
Podemos coletar o `bearer token` da requisição capturada pelo burp-suite e adiciona-la a nossa collection do postman, como mostram as imagens a seguir. **Vale a pena ressaltar que idealmente estas credenciais não seriam adicionadas diretamente às variaveis de ambiente do postman, mas sim guardadas como segredo do vault**, contudo, para fins de demonstração, foram adicionadas diretamente a coleção.

![/Exercicio_2/image.2.2.1.png](/Exercicio_2/image.2.2.1.png)

![/Exercicio_2/image.2.2.2.png](/Exercicio_2/image.2.2.2.png)

Após a configuração do token, podemos testar o acesso a endpoints que aparentam ser exclusivos para admin:


![/Exercicio_2/image.2.2.3.png](/Exercicio_2/image.2.2.3.png)

![/Exercicio_2/image.2.2.4.png](/Exercicio_2/image.2.2.4.png)

![/Exercicio_2/image.2.2.5.png](/Exercicio_2/image.2.2.5.png)

![/Exercicio_2/image.2.2.6.png](/Exercicio_2/image.2.2.6.png)


### Item 2.3
Como podemos verificar nos resultados do item 2.2, a checagem de autorização precisa ocorrer no backend para impedir que agentes maliciosos utilizem seus tokens para utilizar funções aos quais não estão autorizados, como por exemplo funções que lhes permitem criar cupons, produtos e acessar informações de outros usuarios. 

---

# Exercício 3

### Contexto

Fechando a bateria de testes de autorização, a Vetor Security pede um teste sistemático de IDOR (Insecure Direct Object Reference) usando o Burp Intruder, para enumerar automaticamente recursos pertencentes a outros usuários a partir de identificadores sequenciais ou previsíveis. 

Diferente do teste A-B manual do Exercício 1, que confirma a existência da falha em um único par de contas, aqui o objetivo é medir a extensão real do problema: quantos registros de outros usuários estão de fato acessíveis, informação essencial para que o relatório final classifique corretamente a severidade CVSS do achado com base no volume de dados exposto. 

Um único registro vazado e milhares de registros vazados representam a mesma falha técnica, mas geram recomendações de urgência muito diferentes para o cliente.

### Tarefa
1. Configure um ataque no Burp Intruder que itere sobre uma faixa de identificadores de recurso em um endpoint vulnerável a IDOR.
2. Analise as respostas obtidas e identifique quantos recursos de outros usuários foram acessados com sucesso.
3. Proponha uma correção técnica específica para o endpoint testado, apropriada ao padrão de falha identificado.

### Item 3.1
Endpoint escolhido: `GET /identity/api/v2/vehicle/{vehicleId}/location`

Evidencias:

![/Exercicio_3/image.3.1.1.png](/Exercicio_3/image.3.1.1.png)

foram nescessarias remover as seguintes linhas da requisição base para garantir que o servido enviasse os dados dos objetos: 
```
If-Modified-Since: Mon, 22 Sep 2025 18:05:09 GMT
If-None-Match: "68d18fd5-4b7"`
```
Versão atualizada da requisição:

![/Exercicio_3/image.3.1.2.png](/Exercicio_3/image.3.1.2.png)


![/Exercicio_3/image.3.1.3.png](/Exercicio_3/image.3.1.3.png)

### Item 3.2
Podemos analizar com base na evidencia do item anterior que todas as respostas retornaram `HTTP 200` e portanto todos os objetos foram obtidos com sucesso.

### Item 3.3
A seguintes correções podem ser aplicadas no back-end para impedir falhas do tipo IDOR:
1. Utilização de IDs não sequenciais para acessar os objetos
2. Verificação de ownership do objeto, ou seja, verificar se o usuario é autorizado a acessar aquele objeto(Também corrige a falha BOLA verificada na questão 1)
