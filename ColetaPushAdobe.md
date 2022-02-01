# Implementação do rastreamento de Push
Em caso de dúvidas, entrar em contato com: lucas.andrade@bancobmg.com.br ou victor.brunner@bancobmg.com.br

## Objetivo
Este documento tem como objetivo de esclarecer a coleta de interações com as mensagens push, utilizando serviços da FCM, possibilitam. Com foco em **aumentar a visão da área de negócios das métricas de PUSH para usuários do APP.**
Um ponto importante que por hora não existe a possibilidade de coletar a exibição ou envios de push utilizando as ferramentas da Adobe Analytics e Appsflyer. 

## Passos Necessários
* Coletar aberturas do APP via Push
* Enviar parâmetros AF na requisição do Push

## Coletar aberturas do APP via Push
Para habilitar a coleta de aberturas dos pushs. Iremos utilizar a SDK da Appsflyer.

### Configuração Android
Para habilitar esse recurso, chame o método sendPushNotificationData dentro do método onCreate de cada atividade iniciada após clicar na notificação.
```Java
AppsFlyerLib.getInstance().sendPushNotificationData(this);
```

### Configuração iOS
Para habilitar esse recurso, chame o método handlePushNotificationData dentro de AppDelegate.m.
```Objective-C
-(void)application:(UIApplication *)application
    didReceiveRemoteNotification:(NSDictionary *)userInfo
        fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
            [[AppsFlyerLib shared] handlePushNotification:userInfo];
        }
```

## Enviar parâmetros AF na requisição do Push
A AppsFlyer oferece suporte à medição de campanhas de notificação por push de todos os fornecedores e também pode oferecer suporte a desenvolvedores que usam diretamente o Google Cloud Messaging ou os serviços de notificação por push da Apple. 
Os parâmetros a seguir são inseridos no payload da notificação por push. Esses parâmetros devem aparecer em um objeto "af", conforme mostrado nos exemplos abaixo:

| Chave          | Valor               | Detalhe                                                                           | 
|----------------|---------------------|-----------------------------------------------------------------------------------|
| c              | produto-responsavel | Enviar o produto que disparou o push por exemplo: PIX, Transferencia, Cartao, ... | 
| is_retargeting | true                | Sempre enviar true para que a coleta do push seja realizada                       |
| pid            | push-transacional   | Para pushs transacionais sempre enviar push-transacional                          |

Exemplo HTTP V1 API FCM:
```JSON
"data": {
      "af": "{\"pid\":\"push-transacional\",\"is_retargeting\":\"true\", \"c\":\"pix\"}"
  }
```

Os Pushs de campanhas seguem sendo parametrizados pelo time de MKT e enviados para CRM. Cadastrando assim o Deeplink do push com o Onelink parametrizado pela Appsflyer.
Qualquer ponto de dúvida sobre esse cenário atual buscar ajuda com o time de Digital Analytics.
