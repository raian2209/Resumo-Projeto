#  Escalabilidade e Alta Disponibilidade para WordPress na AWS com Elastic Load Balancing e Auto Scaling

## Seção 1: Introdução à Escalabilidade e Alta Disponibilidade na Nuvem AWS

A transição de uma instalação tradicional e monolítica do WordPress para uma implantação moderna e elástica nativa da nuvem representa uma mudança fundamental na arquitetura de aplicações web. Esta seção estabelece os conceitos essenciais para compreender essa transformação, definindo os problemas centrais que os serviços da Amazon Web Services (AWS) são projetados para resolver e introduzindo as ferramentas que formam a base de uma infraestrutura resiliente e de alto desempenho.

### 1.1 O Desafio das Arquiteturas Monolíticas

Uma instalação típica de WordPress em um único servidor, muitas vezes referida como uma arquitetura monolítica, concentra todos os componentes essenciais. Primeiramente, constitui um ponto único de falha (Single Point of Failure): qualquer problema de hardware ou software no servidor pode levar à indisponibilidade total do site. Em segundo lugar, a escalabilidade é limitada. Aumentar a capacidade de um único servidor geralmente requer tempo de inatividade para atualizações de hardware, um processo que não é ágil o suficiente para responder a picos de tráfego repentinos.
Essa abordagem de infraestrutura reflete uma filosofia  onde cada servidor é único, mantido e sua falha é um evento crítico que exige intervenção manual. A nuvem AWS, por outro lado, promove uma mudança para um modelo onde os servidores são idênticos, descartáveis e substituíveis. A resiliência não reside na robustez de uma única instância, mas na capacidade do sistema como um todo de detectar e substituir automaticamente componentes defeituosos, um princípio fundamental para a alta disponibilidade.

### 1.2 Os Pilares de uma Arquitetura Resiliente na AWS

Para implementar a escalabilidade horizontal de forma eficaz, dois serviços principais da AWS são indispensáveis:
- Aplication Load Balancing (ALB): Atua como o ponto de entrada para todo o tráfego da aplicação. Ele distribui automaticamente as solicitações recebidas entre múltiplas instâncias de computação, garantindo que nenhuma delas fique sobrecarregada. Ao direcionar o tráfego apenas para instâncias saudáveis, o ALB elimina pontos únicos de falha e aumenta drasticamente a disponibilidade da aplicação.
- Amazon EC2 Auto Scaling: Este serviço ajusta automaticamente o número de instâncias de computação (Amazon EC2) em resposta à demanda. Durante picos de tráfego, ele adiciona novas instâncias (scale-out) para manter o desempenho. Quando a demanda diminui, ele remove instâncias ociosas (scale-in) para minimizar custos.
A sinergia entre o ALB e o Auto Scaling é o alicerce de uma arquitetura escalável na AWS. O ALB fornece um endereço de rede estável e distribui a carga, enquanto o Auto Scaling gerencia a dinâmica de servidores que respondem a essa carga. Essa combinação não só garante a alta disponibilidade, mas também impulsiona a otimização de custos, um dos principais motivadores para a adoção da nuvem. Portanto, o design das políticas de escalonamento, especialmente as regras de scale-in, torna-se uma decisão de negócio crítica, equilibrando o custo de manter instâncias ociosas com o risco de desempenho degradado.

## Seção 2: Análise Aprofundada do Elastic Load Balancing (ELB)

O Aplication Load Balancing (ALB) é um serviço totalmente gerenciado que serve como o principal ponto de contato para os clientes, distribuindo o tráfego de entrada de forma inteligente e automática para múltiplos destinos, como instâncias Amazon EC2, contêineres e endereços IP, em uma ou mais Zonas de Disponibilidade (Availability Zones).

### 2.1 Fundamentos e Benefícios Essenciais

A função primária de um load balancer é aumentar a disponibilidade e a tolerância a falhas das aplicações. Ao distribuir as cargas de trabalho, o ELB evita que um único servidor se torne um gargalo de desempenho. Ele monitora continuamente a saúde dos seus destinos registrados e encaminha o tráfego apenas para aqueles que estão operando corretamente, isolando e contornando falhas de forma automática. Essa capacidade de adicionar e remover recursos de computação dinamicamente, sem interromper o fluxo geral de solicitações, é fundamental para a elasticidade da arquitetura.

### 2.2 Análise Técnica do Application Load Balancer (ALB)

Diferente de outros tipos de load balancers que operam em camadas mais baixas da rede, o ALB funciona na Camada 7 (camada de aplicação) do modelo OSI, o que o torna "consciente do conteúdo" das solicitações que processa. Essa capacidade permite funcionalidades avançadas de roteamento e gerenciamento de tráfego.

#### Tabela 1: Comparação dos Tipos de Elastic Load Balancer

| Característica        | Application Load Balancer (ALB)                | Network Load Balancer (NLB)                      | Gateway Load Balancer (GLB)                          |
|----------------------|-----------------------------------------------|-------------------------------------------------|-----------------------------------------------------|
| Camada OSI           | Camada 7 (Aplicação)                          | Camada 4 (Transporte)                            | Camada 3 (Rede)                                     |
| Protocolos           | HTTP, HTTPS, gRPC                            | TCP, UDP, TLS                                   | IP (Todos os pacotes)                               |
| Lógica de Roteamento | Baseado em conteúdo (Host, Path, Headers, etc.) | Baseado em conexão (IP, Porta, Protocolo)       | Encaminhamento para Target Group via protocolo GENEVE |
| Tipos de Destino     | Instância EC2, IP, Função Lambda, Contêiner  | Instância EC2, IP, ALB                          | Dispositivos virtuais (firewalls, IDS/IPS)          |
| Caso de Uso Ideal    | Aplicações web, microsserviços, contêineres (Ex: WordPress) | Cargas de trabalho TCP/UDP de altíssimo desempenho e baixa latência | Implantação e escalabilidade de dispositivos de rede de terceiros |

 Sua capacidade de inspecionar o conteúdo das solicitações HTTP permite a implementação de lógicas de roteamento sofisticadas. Por exemplo, seria possível direcionar o tráfego administrativo do WordPress (requisições para /wp-admin/*) para um conjunto específico de servidores otimizados para essas tarefas, enquanto o tráfego público é enviado para um grupo diferente e mais escalável. Essa inteligência permite que o ALB descarregue funcionalidades como terminação SSL e autenticação de usuários, simplificando a arquitetura e tornando as instâncias backend mais enxutas e focadas em servir o conteúdo.

#### 2.2.1 Listeners

Um listener (ouvinte) é um processo que verifica as solicitações de conexão, configurado com um protocolo e uma porta. Para um site WordPress, normalmente são configurados dois listeners: um para HTTP na porta 80 e outro para HTTPS na porta 443. Uma prática recomendada é configurar o listener HTTPS para realizar a "terminação SSL/TLS". Isso significa que o ALB descriptografa as solicitações HTTPS recebidas dos clientes e as encaminha para as instâncias EC2 backend como tráfego HTTP não criptografado. Essa abordagem centraliza o gerenciamento de certificados SSL/TLS no load balancer e alivia as instâncias EC2 da carga computacional da criptografia.

#### 2.2.2 Regras de Roteamento

Cada listener possui regras que determinam como as solicitações são roteadas. O ALB avalia as regras em ordem de prioridade. Uma regra consiste em condições (ex: o caminho da URL é
/blog/*) e ações (ex: encaminhar para o Target Group A). A regra padrão, que não possui condições, é executada se nenhuma outra regra corresponder à solicitação. Essa flexibilidade permite roteamento baseado em host, caminho, cabeçalhos HTTP, métodos de requisição e outros, viabilizando arquiteturas complexas por trás de um único load balancer.

#### 2.2.3 Target Groups

Um Target Group (grupo de destino) é usado para rotear solicitações para um ou mais destinos registrados, como as instâncias EC2 que executam o WordPress. É a ponte entre o load balancer e as instâncias de computação. As instâncias em um Auto Scaling Group são automaticamente registradas e desregistradas deste grupo de destino conforme o grupo escala horizontalmente. O ALB encaminha as solicitações para os destinos saudáveis dentro do grupo.

#### 2.2.4 Verificações de Integridade (Health Checks)

As verificações de integridade são o sistema da alta disponibilidade. O ALB envia periodicamente solicitações para cada destino registrado para testar seu status. Se um destino falhar em um número configurável de verificações consecutivas (
UnhealthyThresholdCount), o ALB o marca como "não saudável" e para de enviar tráfego para ele até que ele passe novamente por um número configurável de verificações bem-sucedidas (HealthyThresholdCount).
A configuração da verificação de integridade é uma decisão arquitetônica crítica. Uma verificação simples (ex: um ping TCP na porta 80) apenas confirma que o servidor web está em execução, mas não detecta problemas na aplicação, como um processo PHP travado ou uma falha de conexão com o banco de dados. Nesse cenário, o ALB continuaria enviando tráfego para uma instância quebrada que retornaria erros 500 aos usuários. A prática recomendada para o WordPress é configurar a verificação de integridade para um endpoint que valide toda a pilha da aplicação, como a página de login /wp-login.php. Uma solicitação para esta página força a execução do PHP e, muitas vezes, uma interação com o banco de dados, fornecendo uma avaliação muito mais precisa da saúde real da instância.

#### 2.2.5 Configurações Necessaria na Aplicação

Quando você coloca um Application Load Balancer na frente das suas instâncias WordPress, o ALB atua como o ponto de terminação SSL/TLS. Isso significa que a conexão segura (HTTPS) do navegador do usuário termina no load balancer. O ALB, por sua vez, se comunica com as instâncias EC2 do WordPress na rede interna da AWS, geralmente por meio de HTTP não criptografado.
Para o WordPress, a solicitação parece vir diretamente do ALB (um endereço IP privado) e usando o protocolo HTTP. Sem uma configuração adequada, isso causa dois problemas principais:
- Geração de URLs Incorretas (Mixed Content): O WordPress, ao gerar URLs para links, folhas de estilo (CSS) e scripts (JavaScript), usará http:// em vez de https://, pois é isso que ele "vê". Isso resulta em erros de "conteúdo misto" (mixed content) nos navegadores dos usuários, quebrando a aparência e a funcionalidade do site.
- Endereço IP do Cliente Incorreto: Funções do WordPress e plugins que dependem do endereço IP do visitante (para comentários, segurança, etc.) verão o IP do load balancer em vez do IP real do usuário.
A Solução: Configurando o wp-config.php
Para resolver isso, você precisa instruir o WordPress a confiar nos cabeçalhos HTTP que o Application Load Balancer encaminhá, que contêm as informações originais da solicitação do cliente. A maneira correta de fazer isso é adicionando um trecho de código ao seu arquivo wp-config.php . Assim configurando o ALB como um proxy reverso.
Todas as instâncias lançadas pelo Auto Scaling estarão preparadas para funcionar corretamente atrás do Application Load Balancer como um proxy reverso, garantindo a integridade e segurança do seu site.

#### 2.2.6 Inicialização Necessaria para ALB

segue o passo a passo de como configurar um Application Load Balancer pelo Console de Gerenciamento da AWS:

- Passo 1: Navegar para a Criação do Load Balancer
  - Acesse o Console de Gerenciamento da AWS.
  - No painel de navegação, encontre a seção EC2.
  - Dentro do painel do EC2, no menu à esquerda, role para baixo até a seção Load Balancing e clique em Load Balancers.
  - Clique no botão Create load balancer.
- Passo 2: Selecionar o Tipo de Load Balancer
  - A AWS oferece diferentes tipos de load balancers. Para um site WordPress, que opera com tráfego HTTP e HTTPS, a escolha correta é o Application Load Balancer (ALB), pois ele opera na camada 7 (aplicação) e pode tomar decisões de roteamento com base no conteúdo da requisição.
  - Na tela de seleção, sob Application Load Balancer, clique em Create.
- Passo 3: Configuração Básica
  - Nesta seção, você definirá as propriedades fundamentais do seu ALB.
  - Load balancer name: Dê um nome descritivo ao seu load balancer (ex: WordPress-ALB). O nome deve ser único na sua região, ter no máximo 32 caracteres e conter apenas letras, números e hifens.
  - Scheme: Selecione Internet-facing. Isso significa que o load balancer receberá tráfego diretamente da internet, o que é necessário para um site público.
  - IP address type: Escolha IPv4. Esta é a opção padrão e suficiente para a maioria dos casos de uso.
- Passo 4: Mapeamento de Rede (Network Mapping)
  - Aqui você define em qual rede virtual (VPC) e sub-redes o seu load balancer irá operar. A alta disponibilidade começa aqui.
  - VPC: Selecione a VPC onde suas instâncias EC2 do WordPress estão ou estarão localizadas.
  - Mappings: Selecione pelo menos duas Zonas de Disponibilidade (Availability Zones). Para cada Zona de Disponibilidade, escolha uma sub-rede pública. Distribuir o load balancer por múltiplas zonas é crucial para garantir a tolerância a falhas; se uma zona ficar indisponível, o ALB nas outras zonas continuará operando.
- Passo 5: Configurar Grupos de Segurança (Security Groups)
  - O Security Group atua como um firewall virtual para o seu load balancer, controlando o tráfego de entrada e saída.
  - Crie um novo Security Group ou selecione um existente.
  - Configure as regras de entrada (Inbound Rules) para permitir tráfego nas portas que seu site usará:
    - HTTP na porta 80 a partir de Anywhere-IPv4 (0.0.0.0/0).
    - HTTPS na porta 443 a partir de Anywhere-IPv4 (0.0.0.0/0).
  - Isso permite que visitantes de qualquer lugar da internet acessem seu site.
- Passo 6: Criar o Target Group (Grupo de Destino)
  - Antes de configurar os "ouvintes" (listeners), você precisa definir para onde o tráfego será enviado. Isso é feito através de um Target Group, que é um agrupamento lógico das suas instâncias EC2 do WordPress.
  - Na seção Listeners and routing, clique no link Create target group. Isso abrirá uma nova aba.
  - Choose a target type: Selecione Instances, pois você registrará instâncias EC2.
  - Target group name: Dê um nome descritivo (ex: WordPress-TG).
  - Protocol / Port: Mantenha HTTP na porta 80. O ALB fará a terminação SSL (HTTPS) e se comunicará com as instâncias via HTTP na rede interna, o que é uma prática comum e eficiente.
  - VPC: Verifique se a mesma VPC do seu ALB está selecionada.
  - Health checks (Verificações de Integridade): Esta é uma configuração crítica.
  - Protocol: Mantenha HTTP.
  - Path: Altere o caminho padrão (/) para um endpoint que valide melhor a saúde da sua aplicação WordPress, como /wp-login.php. Uma resposta bem-sucedida desta (200 ou 304) página confirma que o servidor web e o PHP estão funcionando corretamente.
  - Clique em Next. Na tela seguinte, você pode registrar as instâncias que já estão em execução. Se você ainda não as criou (pois o Auto Scaling fará isso), pode pular este passo e clicar em Create target group.
- Passo 7: Configurar Listeners e Roteamento
  - Volte para a aba de criação do load balancer. Agora você pode configurar os listeners, que são os processos que "escutam" as conexões dos clientes.
  - Listener HTTP (Porta 80):
    - O console já sugere um listener na porta 80.
    - Em Default action, selecione o Target Group que você acabou de criar (WordPress-TG).
- Passo 8: Revisar e Criar
  - Revise todas as configurações na página de resumo para garantir que tudo está correto. Se estiver tudo certo, clique em Create load balancer.

## Seção 3: Análise do Amazon EC2 Auto Scaling

O Amazon EC2 Auto Scaling é o serviço que confere elasticidade à camada de computação da arquitetura, automatizando o provisionamento e o desprovisionamento de instâncias EC2 para corresponder com precisão à demanda da aplicação. Ele garante que o número correto de instâncias esteja sempre disponível para lidar com a carga, otimizando simultaneamente o desempenho e os custos.

### 3.1 Fundamentos e Benefícios Essenciais

O Auto Scaling oferece três benefícios principais que são cruciais para uma arquitetura de nuvem moderna:
Melhor Tolerância a Falhas: O serviço pode detectar quando uma instância se torna "não saudável" (seja por falha em verificações de integridade do EC2 ou do ELB), encerrá-la e iniciar uma nova instância para substituí-la automaticamente. Ao distribuir instâncias em múltiplas Zonas de Disponibilidade, ele pode compensar a indisponibilidade de uma zona inteira.
Melhor Disponibilidade: Garante que a aplicação sempre tenha a capacidade computacional necessária para lidar com a demanda de tráfego atual, evitando a degradação do desempenho durante picos de uso.
Melhor Gerenciamento de Custos: A capacidade é aumentada e diminuída dinamicamente. Como o faturamento do EC2 é baseado no uso, isso resulta em uma economia significativa, pois as instâncias são iniciadas apenas quando necessárias e encerradas quando não são mais.

### 3.2 Anatomia de um Auto Scaling Group (ASG)

O componente central do serviço é o Auto Scaling Group (ASG), uma coleção de instâncias EC2 tratadas como um agrupamento lógico para fins de escalonamento e gerenciamento. A configuração de um ASG envolve vários componentes chave:

#### 3.2.1 Launch Templates

Um Launch Template é um recurso que especifica os parâmetros de configuração para as instâncias EC2 que o ASG irá lançar. Isso inclui o ID da Amazon Machine Image (AMI), tipo de instância, par de chaves, security groups, e quaisquer scripts de inicialização (user data).

#### Tabela 2: Launch Template 

| Característica       | Launch Template                                  |
|---------------------|-------------------------------------------------|
| Versionamento       | Suporta múltiplas versões, permitindo atualizações iterativas |
| Modificabilidade    | Pode ser modificado após a criação (criando uma nova versão) |
| Tipos de Instância Múltiplos | Suporta políticas de instâncias mistas (vários tipos de instância) |
| Opções de Compra Mistas | Suporta uma mistura de instâncias On-Demand e Spot |
| Suporte a Novos Recursos EC2 | Suporte completo para todos os novos recursos do EC2 |
| Recomendação AWS    | Fortemente recomendado para todas as novas implantações |

A capacidade de usar políticas de instâncias mistas, exclusiva dos Launch Templates, é particularmente poderosa. Ela permite que um ASG seja configurado para usar uma combinação de tipos de instância (ex: c5.large, c6g.large) e modelos de compra (On-Demand e Spot). Isso aumenta a disponibilidade, permitindo que o ASG lance um tipo de instância alternativo se o preferencial não estiver disponível, e pode reduzir drasticamente os custos de computação ao aproveitar as instâncias Spot, que são significativamente mais baratas.

#### 3.2.2 Configurações de Capacidade

A configuração de capacidade de um ASG é governada por três parâmetros interligados que funcionam como o sistema de controle central para o escalonamento:
- Tamanho Mínimo (Minimum Size): O número mínimo de instâncias que o ASG deve manter em execução. O grupo nunca escalará para um número abaixo deste limite, garantindo uma capacidade base de serviço.
- Tamanho Máximo (Maximum Size): O número máximo de instâncias que o ASG pode lançar. Este é um limite superior que serve como uma proteção contra escalonamento descontrolado e ajuda a gerenciar os custos.
- Capacidade Desejada (Desired Capacity): A Capacidade Desejada representa o número de instâncias que o ASG tenta manter em um determinado momento. Os limites Mínimo e Máximo são apenas restrições. As ações de escalonamento reais são impulsionadas por mudanças no valor da Capacidade Desejada. Quando uma política de escalonamento é acionada (por exemplo, "a utilização da CPU está alta"), a ação resultante é a de alterar a Capacidade Desejada (ex: de 2 para 3). O loop de controle principal do ASG então detecta a discrepância entre o número real de instâncias em execução (2) e a nova Capacidade Desejada (3) e inicia a ação necessária (lançar uma nova instância) para reconciliar essa diferença. Compreender este mecanismo é vital para diagnosticar e depurar o comportamento de escalonamento.

#### 3.2.3 Integração com o ELB

A integração entre o ASG e o Target Group do ELB é nativa e fundamental para a operação da arquitetura. Ao criar ou atualizar um ASG, ele é associado a um ou mais Target Groups. A partir desse momento, o ciclo de vida das instâncias é gerenciado automaticamente:
- Ao escalar (Scale-out): O ASG lança novas instâncias com base no Launch Template. Assim que uma instância passa nas verificações de integridade iniciais, o ASG a registra automaticamente no Target Group associado. O ELB então começa a enviar tráfego para a nova instância.
- Ao reduzir (Scale-in): O ASG seleciona uma instância para terminação. Primeiro, ele a desregistra do Target Group. O ELB para de enviar novas solicitações para a instância e aguarda o término das solicitações em andamento (um processo chamado connection draining). Somente após o desregistro bem-sucedido, o ASG encerra a instância EC2. Esse processo garante que os usuários não sejam impactados por terminações de instâncias.

#### 3.2.4 Integração passo a passo do Auto Scale Group

Pré-requisitos Essenciais
Antes de criar o Auto Scaling Group, você precisa ter duas coisas prontas:
- Um Launch Template (Modelo de Inicialização): Ele define qual AMI usar , o tipo de instância (ex: t3.micro), o par de chaves, os security groups e qualquer script de inicialização (user data).
- Um Target Group do Application Load Balancer: Este é o grupo de destino que você criou ao configurar o ALB. O Auto Scaling Group precisa saber para qual Target Group ele deve registrar as novas instâncias para que elas possam receber tráfego.

Passo 1: Criar o Launch Template (O Projeto da Instância)
- O Launch Template é o padrão moderno e recomendado pela AWS para definir suas instâncias. Ele substitui as antigas e menos flexíveis Launch Configurations.
- Navegue até o Console EC2: No menu à esquerda, em Instances, clique em Launch Templates.
- Crie o Template: Clique em Create launch template.
- Nome e Descrição:
  - Launch template name: Dê um nome ex: WordPress-Template-V1.
  - Template version description: Descreva a versão ex: Base WordPress com plugin de S3.
- Amazon Machine Image (AMI):
  - Em My AMIs, selecione a sua que você preparou anteriormente. Esta imagem já contém o WordPress configurado e pronto para rodar.
- Instance Type:
  - Escolha um tipo de instância apropriado para o seu site, como t3.micro ou t3.small para começar.
- Key Pair (login):
  - Selecione o par de chaves que você usa para acessar suas instâncias via SSH.
- Network Settings:
  - Subnet: Não selecione uma sub-rede específica aqui. Deixe em branco para que o Auto Scaling Group possa escolher as sub-redes que você definirá para ele mais tarde.
  - Security groups: Selecione o Security Group que você criou para suas instâncias web. Ele deve permitir tráfego do seu Application Load Balancer na porta 80 (HTTP).
- Advanced Details (Detalhes Avançados):
  - IAM instance profile: Se suas instâncias precisam de permissões para interagir com outros serviços da AWS (como o S3 para offload de mídia), selecione o perfil IAM apropriado aqui.
- Crie o Template: Revise as configurações e clique em Create launch template.

Passo 2: Criar o Auto Scaling Group 
- Agora que você tem o projeto (Launch Template), pode criar o grupo que irá gerenciar seus servidores WordPress.
- Navegue até Auto Scaling Groups: No console do EC2, no menu à esquerda, role para baixo até Auto Scaling e clique em Auto Scaling Groups.
- Crie o Grupo: Clique em Create Auto Scaling group.
- Nome e Launch Template:
  - Auto Scaling group name: Dê um nome ex: WordPress-ASG.
  - Launch template: Selecione o WordPress-Template-V1 que você acabou de criar. Clique em Next.
- Configurar Rede:
  - VPC: Escolha a mesma VPC onde seu Application Load Balancer e suas instâncias irão operar.
  - Availability Zones and subnets: Selecione pelo menos duas sub-redes privadas, cada uma em uma Zona de Disponibilidade diferente. Isso é fundamental para a alta disponibilidade. O ASG irá distribuir as instâncias entre essas zonas.
- Anexar o Load Balancer:
  - Selecione Attach to an existing load balancer.
  - Escolha Choose from your load balancer target groups.
  - Selecione o Target Group que você criou para o seu WordPress (ex: WordPress-TG).
- Health checks: Marque a caixa ELB em Health check type. Isso instrui o Auto Scaling a considerar uma instância como "não saudável" se o Application Load Balancer reportar que ela falhou nas verificações de integridade. É uma camada extra de resiliência.
- Configurar Tamanho do Grupo e Políticas de Escalonamento:
  - Group size: Aqui você define os limites de capacidade.
  - Desired capacity (Capacidade desejada): O número de instâncias com que o grupo deve começar e tentar manter. Um bom ponto de partida é 2 para alta disponibilidade.
  - Minimum capacity (Capacidade mínima): O número mínimo de instâncias. O grupo nunca terá menos que isso. Defina como 2.
  - Maximum capacity (Capacidade máxima): O limite superior de instâncias. Defina como 4 (ou mais, dependendo da sua necessidade).
  - Scaling policies: Aqui você define a lógica de escalonamento.
  - Selecione Target tracking scaling policy.
  - Metric type: Escolha Average CPU utilization.
  - Target value: Digite 75 (%).
  - Deixe as outras opções como padrão. Esta única política cuidará tanto do scale-out (quando a CPU passar de 75%) quanto do scale-in (quando a CPU ficar ociosa).8
- Adicionar Notificações (Opcional, mas Recomendado):
  - Você pode configurar notificações via Amazon SNS para ser alertado por e-mail sempre que o ASG lançar ou terminar uma instância.
- Revisar e Criar:
  - Revise todas as configurações e, se estiver tudo correto, clique em Create Auto Scaling group.

Passo 3: Verificar a Inicialização
- Após a criação, o Auto Scaling Group começará a trabalhar imediatamente.
- Monitore as Instâncias: Vá para o painel de Instances do EC2. Você verá novas instâncias sendo iniciadas com o nome (tag) que você definiu. O número de instâncias deve corresponder à sua "Desired Capacity".
- Verifique o Target Group: Vá para Target Groups, selecione seu WordPress-TG e clique na aba Targets. As novas instâncias aparecerão aqui. Inicialmente, o status será initial. Após alguns instantes, se tudo estiver configurado corretamente, o status mudará para healthy.
- Acesse seu Site: Use o DNS do seu Application Load Balancer para acessar o site. Ele agora está sendo servido por uma frota de instâncias resiliente e pronta para escalar.
Com esses passos, seu Auto Scaling Group está inicializado e totalmente operacional, garantindo que seu site WordPress tenha os recursos necessários para lidar com o tráfego, mantendo o desempenho e otimizando os custos.9

## Seção 4: Políticas de Escalonamento: A Lógica de Scale-Up e Scale-Down

As políticas de escalonamento são as políticas da arquitetura elástica, definindo as regras e condições que governam quando e como o Auto Scaling Group deve adicionar (scale-out) ou remover (scale-in) instâncias. A escolha da política correta é uma decisão que equilibra o risco de degradação de desempenho com o custo de manter recursos ociosos.

### 4.1 Visão Geral das Estratégias de Escalonamento

A AWS oferece várias estratégias para automatizar o escalonamento, que podem ser usadas isoladamente ou em combinação para criar uma resposta robusta  às mudanças na demanda :
- Escalonamento Dinâmico: Reage em tempo real a mudanças nas métricas de desempenho, como utilização de CPU ou tráfego de rede.
- Escalonamento Agendado: Atua em horários predefinidos, ideal para lidar com picos de tráfego previsíveis.
- Escalonamento Preditivo: Usa machine learning para prever a demanda futura com base em dados históricos e provisionar capacidade proativamente.

#### Tabela 3: Comparação das Políticas de Escalonamento Dinâmico

| Característica         | Target Tracking Scaling   | Step Scaling              |
|-----------------------|---------------------------|---------------------------|
| Complexidade de Configuração | Simples                | Média/Complexa             |
| Granularidade de Controle | Baixa (Define um alvo)     | Alta (Múltiplos passos e ajustes) |
| Caso de Uso Principal  | Manter uma métrica de utilização estável (ex: CPU em 50%) | Lidar com picos de tráfego com respostas de magnitude variável |
| Gerenciamento de Alarmes | Gerenciado pela AWS       | Gerenciado pelo usuário    |
| Recomendação          | Preferencial para a maioria dos casos de uso | Para controle granular e respostas agressivas a grandes picos |

### 4.2 Escalonamento Dinâmico - Target Tracking

Para a maioria das aplicações, incluindo WordPress, o Target Tracking Scaling (escalonamento com monitoramento de objetivo) é a abordagem mais simples e eficaz. O conceito é análogo ao de um termostato: o usuário define uma métrica-alvo (ex: utilização média de CPU) e um valor-alvo (ex: 50%). Por exemplo, ao configurar uma política de Target Tracking para manter a CPUUtilization média em 50%:
- Se a carga da aplicação aumentar e a CPU média subir para 60%, o Auto Scaling iniciará uma ação de scale-out, adicionando instâncias para reduzir a utilização de volta para perto de 50%.
- Se a carga diminuir e a CPU média cair para, digamos, 25%, o Auto Scaling iniciará uma ação de scale-in, removendo instâncias para aumentar a utilização das restantes e reduzir custos.
É crucial entender a assimetria a essa política: ela é projetada para ser mais agressiva ao adicionar capacidade (scale-out) e mais conservadora ao removê-la (scale-in). Isso prioriza a disponibilidade, evitando que o sistema remova instâncias prematuramente durante uma calmaria temporária no tráfego, o que poderia levar a oscilações rápidas.

### 4.3 Escalonamento Dinâmico - Step Scaling

Para cenários que exigem um controle mais granular, o Step Scaling (escalonamento por etapas) é a melhor opção. Em vez de um único alvo, o Step Scaling permite definir uma série de ajustes de escalonamento que variam com base na magnitude da violação do alarme.
Por exemplo, uma política de scale-out pode ser definida da seguinte forma :
- Se a CPU média estiver entre 60% e 75%, adicione 1 instância.
- Se a CPU média estiver entre 75% e 90%, adicione 3 instâncias.
- Se a CPU média for superior a 90%, adicione 5 instâncias.
Essa abordagem permite uma resposta proporcional, aplicando uma ação modesta para pequenos aumentos de carga e uma ação muito mais agressiva para picos repentinos e intensos. As políticas de scale-in podem ser configuradas de forma semelhante. Os ajustes podem ser definidos como um número específico de instâncias (ChangeInCapacity), uma porcentagem da frota atual (PercentChangeInCapacity) ou um número exato de capacidade (ExactCapacity).

### 4.4 Escalonamento Agendado

O Scheduled Scaling (escalonamento agendado) é uma ferramenta proativa para lidar com padrões de tráfego conhecidos e previsíveis. Em vez de reagir a métricas, ele executa ações de escalonamento em horários específicos, seja como um evento único ou em um cronograma recorrente (usando expressões cron).
Um caso de uso clássico para um site de e-commerce WordPress é uma promoção de Black Friday. Uma ação agendada pode ser criada para aumentar os valores de Min Size e Desired Capacity uma hora antes do início da promoção, garantindo que a capacidade extra esteja pronta e aquecida antes da chegada do pico de tráfego. Outra ação agendada pode reduzir a capacidade após o término do evento, otimizando os custos.

### 4.5 Parâmetros Críticos de Escalonamento: Cooldowns e Warmups

Para garantir que o escalonamento automático funcione de forma suave e estável, dois parâmetros são essenciais:
- Cooldowns de Escalonamento: Um período de cooldown define um tempo de espera após a conclusão de uma atividade de escalonamento antes que outra possa ser iniciada. Isso evita que o sistema reaja exageradamente a flutuações rápidas e temporárias nas métricas, um comportamento conhecido como "thrashing". Por exemplo, após um scale-out, o cooldown dá tempo para que as novas instâncias comecem a assumir a carga e a métrica de CPU se estabilize antes de tomar outra decisão de escalonamento.
- Aquecimento da Instância (Instance Warmup): O parâmetro DefaultInstanceWarmup informa ao Auto Scaling por quanto tempo uma instância recém-lançada deve ser ignorada ao calcular as métricas agregadas do grupo (como a CPU média). Uma nova instância pode apresentar alta utilização de CPU durante a inicialização e configuração. Sem um período de warmup, essa alta utilização poderia influenciar indevidamente a métrica agregada, potencialmente desencadeando outra ação de scale-out desnecessária. O warmup garante que apenas instâncias totalmente operacionais e estabilizadas contribuam para as decisões de escalonamento.

### 4.6 Inicialização Passo a passo de políticas


- Passo 1: Criar o Alarme do CloudWatch para Scale-Up (CPU Alta)
  - Primeiro, precisamos criar o "gatilho" que informará ao Auto Scaling Group que a carga está alta. Esse gatilho é um alarme do Amazon CloudWatch.
  - Navegue até o Console do CloudWatch.
  - No menu à esquerda, clique em Alarms e depois em All alarms.
  - Clique em Create alarm e depois em Select metric.
  - Na aba All metrics, escolha EC2 e depois By Auto Scaling Group.
  - Encontre e selecione a métrica CPUUtilization para o seu Auto Scaling Group do WordPress.
  - Na seção Conditions, configure o seguinte:
  - Threshold type: Static.
  - Whenever CPUUtilization is...: Selecione Greater (>).
  - than...: Digite 75.
  - Em Additional configuration:
  - Datapoints to alarm: Define como 2 por 5 Minutes. Isso significa que o alarme só será acionado se a CPU média permanecer acima de 75% por 10 minutos consecutivos, evitando reações a picos de tráfego muito curtos.
  - Clique em Next.
  - Na página Configure actions, não precisa adicionar uma notificação. Apenas clique em Next.
  - Dê um nome ao alarme, como WordPress-CPU-High-ScaleUp, e clique em Next.
  - Revise e clique em Create alarm.
- Passo 2: Criar o Alarme do CloudWatch para Scale-Down (CPU Baixa)
  - Agora, vamos criar o gatilho para remover instâncias quando a carga estiver baixa.
  - Siga os mesmos passos de 1 a 5 da seção anterior para selecionar a métrica CPUUtilization do seu Auto Scaling Group.
  - Na seção Conditions, configure o seguinte:
  - Threshold type: Static.
  - Whenever CPUUtilization is...: Selecione Lower (<).
  - than...: Digite 25.
  - Em Additional configuration:
  - Datapoints to alarm: Defina como 3 por 5 Minutes. É uma boa prática ser mais conservador ao remover instâncias. Isso garante que a CPU permaneça ociosa por 15 minutos antes de uma instância ser desligada, confirmando que a queda no tráfego é sustentada.
  - Clique em Next.
  - Na página Configure actions, não adicione notificações e clique em Next.
  - Dê um nome ao alarme, como WordPress-CPU-Low-ScaleDown, e clique em Next.
  - Revise e clique em Create alarm.
- Passo 3: Criar a Política de Scale-Up (Adicionar Instância)
  - Com os alarmes criados, agora vamos dizer ao Auto Scaling Group o que fazer quando o alarme de CPU alta for acionado.
  - Navegue até o Console do EC2 e, no menu à esquerda, vá para Auto Scaling Groups.
  - Selecione o seu Auto Scaling Group do WordPress.
  - Vá para a aba Automatic scaling.
  - Em Dynamic scaling policies, clique em Create dynamic scaling policy.
  - Configure a política da seguinte forma:
  - Policy type: Step scaling.
  - Scaling policy name: Dê um nome, como Add-One-Instance-High-CPU.
  - CloudWatch alarm: Selecione o alarme WordPress-CPU-High-ScaleUp que você criou no Passo 1.
  - Take the action:
  - Selecione Add.
  - Digite 1.
  - Selecione capacity units.
  - Clique em Create.
  - Como funciona: Quando o alarme WordPress-CPU-High-ScaleUp for acionado (CPU > 75%), esta política será executada e aumentará a "Capacidade Desejada" do seu grupo em 1, fazendo com que uma nova instância seja iniciada.
- Passo 4: Criar a Política de Scale-Down (Remover Instância)
  - Finalmente, vamos configurar a regra para remover uma instância quando o alarme de CPU baixa for acionado.
  - Ainda na aba Automatic scaling do seu Auto Scaling Group, clique novamente em Create dynamic scaling policy.
  - Configure a política da seguinte forma:
  - Policy type: Step scaling.
  - Scaling policy name: Dê um nome, como Remove-One-Instance-Low-CPU.
  - CloudWatch alarm: Selecione o alarme WordPress-CPU-Low-ScaleDown que você criou no Passo 2.
  - Take the action:
  - Selecione Remove.
  - Digite 1.
  - Selecione capacity units.
  - Clique em Create.
  - Como funciona: Quando o alarme WordPress-CPU-Low-ScaleDown for acionado (CPU < 25%), esta política será executada e diminuirá a "Capacidade Desejada" do seu grupo em 1. O Auto Scaling Group então usará sua política de terminação para escolher e desligar uma instância.

### Resumo da Inicialização

Ao concluir esses quatro passos, seu sistema estará totalmente automatizado:
- Scale-Up (Aumentar): Se a utilização média da CPU em todas as suas instâncias ultrapassar 75% por um período sustentado, o primeiro alarme dispara, a política de scale-up é acionada e uma nova instância do WordPress é automaticamente adicionada ao grupo e registrada no seu Application Load Balancer.
- Scale-Down (Diminuir): Se o tráfego diminuir e a utilização média da CPU cair abaixo de 25% por um período sustentado, o segundo alarme dispara, a política de scale-down é acionada e uma instância é removida do grupo para otimizar os custos.
