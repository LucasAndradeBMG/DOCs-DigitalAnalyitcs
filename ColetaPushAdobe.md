# Implementação do rastreamento de push Adobe + Appsflyer
Em caso de dúvidas, entrar em contato com: lucas.andrade@bancobmg.com.br ou victor.brunner@bancobmg.com.br

## Objetivo
Este documento tem como objetivo de esclarecer a coleta clicks throughs e demais interações que as mensagens push, utilizando serviços da FCM, possibilitam. Com foco em **aumentar a visão da área de negócios das métricas de SMS e PUSH para usuários do APP.**
Um ponto importante que por hora não existe a possibilidade de coletar a exibição ou envios de push utilizando as ferramentas da Adobe Analytics e Appsflyer. 

* Configurar Ferramentas Adobe e Appsflyer para realizar a coleta (Responsabilidade time Digital Analytics)
* Obter o ID/token de registro usando o FCM
* Enviar parâmetros de mídia no push
* Coletar aberturas Push

## Obter o ID/token de registro usando o FCM
Esse passo é necessário exclusivamente para disponibilizar a métrica de desinstalação é necessária a coleta do Token do FCM. 
Além de coletar e enviar para a SDK Adobe gostaríamos que fosse enviado o Token para a ferramenta Adobe Analytics. 
Podendo ser enviado na variável **appInfo_tokenFCM** e disparado posteriormente em qualquer evento **MobileCore.trackState** para ser registrado no Adobe Analytics.

É recomendável chamar setPushIdentifier em cada inicialização do aplicativo para garantir que o token de dispositivo mais atualizado seja definido para o SDK. Se nenhum token de dispositivo estiver disponível, null/nil deve ser passado. Enviar o token para a SDK Adobe como nos exemplos abaixo.

### Configuração Android

```Java
import com.appsflyer.AppsFlyerLib;
import com.google.firebase.messaging.FirebaseMessagingService;

public class MyNewFirebaseManager extends FirebaseMessagingService {

    @Override
    public void onNewToken(String token) {
        super.onNewToken(token);

        // Enviar um novo token para Appsflyer
       	AppsFlyerLib.getInstance().updateServerUninstallToken(getApplicationContext(), token);
        // Gravar valor de um novo token para Adobe Analytics 
        Map<String, String> additionalContextData = new HashMap<String, String>();
        additionalContextData.put("appInfo_tokenFCM", token);        
    }
}
```


### Configuração iOS
É necessário registrar as notificações push no nível de código do aplicativo para permitir a coleta de dados.
Adicione o seguinte código ao seu AppDelegate.m:

```Objective-C
- #import <UserNotifications/UserNotifications.h>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // O userNotificationTypes abaixo é apenas um exemplo e pode ser alterado de acordo com o APP
 UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
          center.delegate = self;
          [center requestAuthorizationWithOptions:(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge) completionHandler:^(BOOL granted, NSError * _Nullable error) {
          }];
   [[UIApplication sharedApplication] registerForRemoteNotifications];
    
  }
  - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    // Enviar um novo token para Appsflyer
    [[AppsFlyerLib shared] registerUninstall:deviceToken];
    // Gravar valor de um novo token para Adobe Analytics
    NSMutableDictionary *contextData = [NSMutableDictionary dictionary]; 
    [contextData setObject:@deviceToken forKey:@"appInfo_tokenFCM"]; 
  }
  }
```

Em AppDelegate.h, insira o código a seguir para adicionar UNUserNotificationCenterDelegate à declaração da interface:

```Objective-C
#import <AppsFlyerLib/AppsFlyerLib.h>
@interface AppDelegate : UIResponder <UIApplicationDelegate, AppsFlyerLibDelegate>
```
## Enviar parâmetros de mídia no push


## Coletar aberturas do APP via Push
 Cada abertura de push irá ser tratada como um evento customizado do Adobe Analytics ('trackAction'). Ou seja quando o APP abrir vindo de uma notificação ele deverá disparar um trackAction com os seguintes valores abaixo:

|  Chave | Valor  |  
|---     |---     |
|&&channel |Institucional| 
|screenInfo_name|'bmg|app|nl|na|institucional|home'|
|sessionInfo_user_status|'NL'|
|appInfo_brand|'Meu BMG'|
|sessionInfo_content_message|BMG Card - Juros mais baixo que cartões comuns e sem anuidade|

Concatenar titulo e texto do Push para a valor da variável sessionInfo_content_message. Separar titulo do texto com o carácter -

### Configuração Android
```Java
    Map<String, String> additionalContextData = new HashMap<String, String>();
    additionalContextData.put("sessionInfo_content_message", "BMG Card - Juros mais baixo que cartões comuns e sem anuidade");
    //disparar um trackAction para Push, com a camada de dados contendo a chave sessionInfo_content_message com o titulo e texto do push. 
    MobileCore.trackAction("Push", additionalContextData);	
} 
```

### Configuração iOS
```Objective-C
    var additionalContextData = new NSMutableDictionary<NSString, NSString>
    additionalContextData setObject:@"BMG Card - Juros mais baixo que cartões comuns e sem anuidade" forKey:@"sessionInfo_content_message"];    
    //disparar um trackAction para Push, com a camada de dados contendo a chave sessionInfo_content_message com o titulo e texto do push. 
    [ACPCore trackAction:@"Push" ,  data:additionalContextData];
} 
```
