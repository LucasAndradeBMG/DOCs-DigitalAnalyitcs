# Mensuração de Desintalação APP utilizando Appsflyer
Em caso de dúvidas, entrar em contato com: lucas.andrade@bancobmg.com.br ou lucas.hocihara@bancobmg.com.br

## Objetivo
Este documento tem como objetivo de esclarecer a coleta de uma métrica de desinstalação utilizando serviços da FMC e Appsflyer. Com foco em **aumentar a visão da área de negócios sobre desinstalações.**

## Configuração Android
Pretendemos trazer um resumo de como realizar a configuração no Android. [Para acessar a documentação completa acessar aqui](https://support.appsflyer.com/hc/pt/articles/360017822118-Fa%C3%A7a-a-integra%C3%A7%C3%A3o-da-mensura%C3%A7%C3%A3o-de-desinstala%C3%A7%C3%A3o-do-Android-a-um-aplicativo)


* Ter o FCM configurado no APP
* Configurar na Appsflyer a coleta (Responsabilidade time Digital Analytics)
* Integrar FCM a Appsflyer e enviar Token para Appsflyer e Adobe

### Integrar FCM a Appsflyer e enviar Token para Appsflyer e Adobe
Para disponibilizar a métrica de desisntalação é necessária a coleta do Token do FMC. 
Além de coletar e enviar para a Appsflyer gostariamos que fosse enviado o Token para a ferramenta Adobe Analytics. Podendo ser enviado na variavel **appInfo_tokenFMC** e disparado posteriormente em qualquer evento **MobileCore.trackState** para ser registrado no Adobe Analytics.

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
               additionalContextData.put("appInfo_tokenFMC", token);        
    }
}
```

## Configuração iOS
Pretendemos trazer um resumo de como realizar a configuração no iOS. [Para acessar a documentação completa acessar aqui](https://support.appsflyer.com/hc/pt/articles/360017822178-Fa%C3%A7a-a-integra%C3%A7%C3%A3o-da-m%C3%A9trica-de-desinstala%C3%A7%C3%A3o-do-iOS-a-um-aplicativo-)

* Gerar e exportar certificado p12 para time de Digital Analytcs
* Configurar na Appsflyer a coleta (Responsabilidade time Digital Analytics)
* Garantir que temos permissões para notificações push em segundo plano no APP

### Permitir a coleta no APP
Para disponibilizar a métrica de desisntalação é necessária a coleta do Token do FMC. 
Além de coletar e enviar para a Appsflyer gostariamos que fosse enviado o Token para a ferramenta Adobe Analytics. Podendo ser enviado na variavel **appInfo_tokenFMC** e disparado posteriormente em qualquer evento **MobileCore.trackState** para ser registrado no Adobe Analytics.

```Swift
//add UserNotifications.framework
	import UserNotifications
	func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    //...
    // iOS 10 support
    if #available(iOS 10, *) {
      UNUserNotificationCenter.current().requestAuthorization(options:[.badge, .alert, .sound]){ (granted, error) in }
      application.registerForRemoteNotifications()
    }
    // iOS 9 and iOS 8 support
    else if #available(iOS 8, *), #available(iOS 9, *) {
      UIApplication.shared.registerUserNotificationSettings(UIUserNotificationSettings(types: [.badge, .sound, .alert], categories: nil))
      UIApplication.shared.registerForRemoteNotifications()
    }

    // iOS 7 support
    else {
      application.registerForRemoteNotifications(matching: [.badge, .sound, .alert])
    }
    
    return true
  }

 // Called when the application sucessfuly registers for push notifications
  func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    AppsFlyerLib.shared().registerUninstall(deviceToken)
    //Gravar valor do Token para posteriormente enviar nos disparos ACPCore.trackState do Adobe Analytics 
    NSMutableDictionary *contextData = [NSMutableDictionary dictionary]; 
    [contextData setObject:deviceToken forKey:@"appInfo_tokenFMC"]; 
  }
```

Os trechos de código acima solicitam permissões do usuário para enviar notificações push. No entanto, se você estiver usando apenas notificações push em segundo plano para medir desinstalações, não há necessidade de solicitar a permissão do usuário. Isso ocorre porque essas notificações push são notificações push silenciosas que [não precisam pedir a permissão dos usuários](https://developer.apple.com/videos/play/wwdc2015/720/?time=126).