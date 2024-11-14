# O que iremos ver hoje?

Hoje, conheceremos profundamente o mundo do RBAC (Role-Based Access Control) no Kubernetes.
O RBAC é um mecanismo essencial que permite a administradores de sistemas definirem regras de acesso específicas para usuários e
serviços dentro de um cluster Kubernetes. Você aprenderá sobre a importância do RBAC para a segurança e a gestão eficaz dos recursos de um cluster,
incluindo como criar e gerenciar usuários, atribuir papéis e permissões específicas, e configurar acessos baseados em funções para diferentes tipos de usuários e serviços.
Vamos explorar exemplos práticos de como aplicar o RBAC para controlar o acesso a recursos do cluster, como pods, deployments e serviços, garantindo que apenas usuários autorizados
possam executar operações específicas. Ao final deste guia, você terá um entendimento sólido sobre como implementar e gerenciar o controle de acesso baseado em funções no Kubernetes,
proporcionando um ambiente de cluster mais seguro e eficiente.

## RBAC

O que é RBAC?
RBAC é um acrônimo para Role-Based Access Control, ou Controle de Acesso Baseado em Funções. É um método de controle de acesso que permite que um administrador defina permissões específicas para usuários e grupos de usuários. Isso significa que os administradores podem controlar quem tem acesso a quais recursos e o que eles podem fazer com esses recursos.

No Kubernetes é super importante você entender como funciona o RBAC, pois é através dele que você vai definir as permissões de acesso aos recursos do cluster, como por exemplo, quem pode criar um Pod, um Deployment, um Service, etc.

Primeiro exemplo de RBAC
Vamos imaginar que precisamos dar acesso ao cluster a uma pessoa desenvolvedora da nossa empresa, mas não queremos que ela tenha acesso a todos os recursos do cluster, apenas aos recursos que ela precisa para desenvolver a sua aplicação.

Para isso, vamos criar um usuário chamado developer e vamos dar acesso a ele para criar e administrar os Pods no namespace dev.

Temos duas formas de fazer isso, a primeira e mais antiga é através da criação de um Token de acesso, e a que iremos abordar na sequência é
através da criação de um certificado. O Token é mais utilizado para dar acesso a um ServiceAccount, que é um usuário que não é humano.
Por exemplo, podemos ter um ServiceAccount para o Prometheus poder coletar métricas do cluster, ou um ServiceAccount para o Fluentd poder coletar
os logs do cluster. E podemos ter um User para um usuário humano, como por exemplo, o usuário developer que iremos criar.
