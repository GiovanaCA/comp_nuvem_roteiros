# Criando um plano de Disaster Recovery e SLA

**Você é o CTO (Chief Technology Officer) de uma grande empresa com sede em várias capitais no Brasil e precisa implantar um sistema crítico, de baixo custo e com dados sigilosos para a área operacional.**

**A. Você escolheria Public Cloud ou Private Cloud?**

Para as tarefas de processamento, seria beneficial escolher uma Public Cloud, devido à alta disponibilidade e escalabilidade. E para as tarefas que requerem maior segurança seria melhor uma Private Cloud, proporcionando um  maior controle da segurança de dados sigilosos e uma baixa latência para trabalhos. Logo, uma boa escolha seria fazer uma Nuvem Híbrida, possuindo boa segurança e flexibilidade, mas vale-se mencionar que é complexo integrar e gerenciar os ambientes das duas nuvens eficientemente e que os funcionários devem possuir a habilidade de lidar com os dois ambientes. 

**B. Agora explique para ao RH por que você precisa de um time de DevOps.**

Um time de DevOps é necessário, uma vez que, além de fornecer uma  grande escalabilidade e eficiência, ele seria responsável por atualizações e melhorias periódicas, pela automação de processos e pelo monitoramento, rapidamente identificando problemas e implementando soluções.

**C. Considerando o mesmo sistema crítico, agora sua equipe deverá planejar e implementar um ambiente resiliente e capaz de mitigar  possíveis interrupções/indisponibilidades. Para isso, identifiquem quais são as principais ameaças que podem colocar sua infraestrutura em risco, e descreva as principais ações que possibilitem o restabelecimento de todas as aplicações de forma rápida e organizada caso algum evento cause uma interrupção ou incidente de segurança. Para isso monte um plano de DR e HA que considere entre as ações:**

-  **Mapeamento das principais ameaças que podem colocar em riscos o seu ambiente.**
-  **Elenque e priorize as ações para a recuperação de seu ambiente em uma possível interrupção/desastre.**
-  **Como sua equipe irá tratar a política de backup?**
-  **Considerando possíveis instabilidades e problemas, descreva como alta disponibilidade será implementada em sua infraestrutura.**

Ameaças:

-  Ataques hackers, falhas de hardware, erro humano e instabilidade de rede. 

Recuperação de ambiente:

-  Backup automatizado e servidores redundantes. 

Política de backup:

-  Frequente, criptografia e armazenar em mais de um servidor. 

Alta disponibilidade:

-  Load balancer e estratégia de multi-cloud.