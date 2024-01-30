# Esp32_Now

## Introduction
ESP-NOW est une sorte de protocole de communication Wi-Fi sans connexion défini par Espressif. Dans ESP-NOW, les données d'application sont encapsulées dans un cadre d'action spécifique au fournisseur, puis transmises d'un appareil Wi-Fi à un autre sans connexion.

## Format du cadre
ESP-NOW utilise un cadre d'action spécifique au fournisseur pour transmettre les données ESP-NOW. Le débit binaire ESP-NOW par défaut est de 1 Mbps. Le format du cadre d'action spécifique au fournisseur est le suivant :

```
------------------------------------------------------------------------------------------------------------
| MAC Header | Category Code | Organization Identifier | Random Values | Vendor Specific Content |   FCS   |
------------------------------------------------------------------------------------------------------------
  24 bytes         1 byte              3 bytes               4 bytes             7-257 bytes        4 bytes
```

Contenu spécifique au fournisseur : le contenu spécifique au fournisseur contient les champs spécifiques au fournisseur comme suit :

```
-------------------------------------------------------------------------------
| Element ID | Length | Organization Identifier | Type | Version |    Body    |
-------------------------------------------------------------------------------
    1 byte     1 byte            3 bytes         1 byte   1 byte   0-250 bytes
```

## Sécurité

ESP-NOW utilise la méthode CCMP, décrite dans IEEE Std. 802.11-2012, pour protéger le cadre d'action spécifique au fournisseur. Le périphérique Wi-Fi conserve une clé principale primaire (PMK) et plusieurs clés principales locales (LMK). Les longueurs de PMK et LMk sont de 16 octets.

PMK est utilisé pour chiffrer LMK avec l'algorithme AES-128. Appelez ```esp_now_set_pmk()``` pour définir PMK. Si PMK n’est pas défini, un PMK par défaut sera utilisé.

Le LMK de l'appareil couplé est utilisé pour chiffrer la trame d'action spécifique au fournisseur avec la méthode CCMP. Le nombre maximum de LMK différents est de six. Si le LMK de l'appareil couplé n'est pas défini, le cadre d'action spécifique au fournisseur ne sera pas chiffré.

Le chiffrement de la trame d’action spécifique au fournisseur de multidiffusion n’est pas pris en charge.

## Initialisation et désinitialisation

Appelez ```esp_now_init()``` pour initialiser ESP-NOW et ```esp_now_deinit()``` pour désinitialiser ESP-NOW. Les données ESP-NOW doivent être transmises après le démarrage du Wi-Fi, il est donc recommandé de démarrer le Wi-Fi avant d'initialiser ESP-NOW et d'arrêter le Wi-Fi après avoir désinitialisé ESP-NOW.

Lorsque ```esp_now_deinit()``` est appelé, toutes les informations des appareils couplés sont supprimées.

## Ajouter un appareil couplé

Appelez ```esp_now_add_peer()``` pour ajouter l'appareil à la liste des appareils couplés avant d'envoyer des données à cet appareil. Si la sécurité est activée, le LMK doit être défini. Vous pouvez envoyer des données ESP-NOW via la Station et l'interface SoftAP. Assurez-vous que l'interface est activée avant d'envoyer des données ESP-NOW.

Le nombre maximum d'appareils couplés est de 20 et les appareils de cryptage couplés ne dépassent pas 17, la valeur par défaut est 7. Si vous souhaitez modifier le nombre d'appareils de cryptage couplés, définissez CONFIG_ESP_WIFI_ESPNOW_MAX_ENCRYPT_NUM dans le menu de configuration des composants Wi-Fi.

Un appareil avec une adresse MAC de diffusion doit être ajouté avant d'envoyer des données de diffusion. La plage du canal des appareils couplés est de 0 à 14. Si le canal est réglé sur 0, les données seront envoyées sur le canal actuel. Sinon, le canal doit être défini comme étant celui sur lequel se trouve le périphérique local.

## Envoyer des données ESP-NOW

Appelez ```esp_now_send()``` pour envoyer des données ESP-NOW et ```esp_now_register_send_cb()``` pour enregistrer la fonction de rappel d'envoi. Il renverra ESP_NOW_SEND_SUCCESS lors de l'envoi de la fonction de rappel si les données sont reçues avec succès sur la couche MAC. Sinon, il renverra ESP_NOW_SEND_FAIL. Plusieurs raisons peuvent amener ESP-NOW à ne pas envoyer de données. Par exemple, le périphérique de destination n'existe pas ; les canaux des appareils ne sont pas les mêmes ; la trame d'action est perdue lors de la transmission à l'antenne, etc. Il n'est pas garanti que la couche application puisse recevoir les données. Si nécessaire, renvoyez les données d'accusé de réception lors de la réception des données ESP-NOW. Si la réception des données d'accusé de réception expire, retransmettez les données ESP-NOW. Un numéro de séquence peut également être attribué aux données ESP-NOW pour supprimer les données en double.

S'il y a beaucoup de données ESP-NOW à envoyer, appelez ```esp_now_send()``` pour envoyer moins ou égal à 250 octets de données une fois. Notez qu'un intervalle trop court entre l'envoi de deux données ESP-NOW peut entraîner un dysfonctionnement de la fonction de rappel d'envoi. Il est donc recommandé d'envoyer les prochaines données ESP-NOW après le retour de la fonction de rappel d'envoi de l'envoi précédent. La fonction de rappel d'envoi s'exécute à partir d'une tâche Wi-Fi hautement prioritaire. Alors, n’effectuez pas d’opérations longues dans la fonction de rappel. Au lieu de cela, publiez les données nécessaires dans une file d'attente et gérez-les à partir d'une tâche de priorité inférieure.

## Réception de données ESP-NOW

Appelez ```esp_now_register_recv_cb()``` pour enregistrer la fonction de rappel de réception. Appelez la fonction de rappel de réception lors de la réception d'ESP-NOW. La fonction de rappel de réception s'exécute également à partir de la tâche Wi-Fi. Alors, n’effectuez pas d’opérations longues dans la fonction de rappel. Au lieu de cela, publiez les données nécessaires dans une file d'attente et gérez-les à partir d'une tâche de priorité inférieure.

## Configuration du taux ESP-NOW

Appelez ```esp_wifi_config_espnow_rate()``` pour configurer le débit ESP-NOW de l'interface spécifiée. Assurez-vous que l'interface est activée avant le taux de configuration. Cette API doit être appelée après ```esp_wifi_start()```.

## Paramètre d'économie d'énergie de configuration ESP-NOW

La veille est prise en charge uniquement lorsque l'ESP32 est configuré en tant que station.

Appelez ```esp_now_set_wake_window()``` pour configurer Window pour ESP-NOW RX en veille. La valeur par défaut est le maximum, qui autorise RX à tout moment.

Si une économie d'énergie est nécessaire pour ESP-NOW, appelez ```esp_wifi_connectionless_module_set_wake_interval()``` pour configurer également l'intervalle.

## Header File

Ce fichier d'en-tête peut être inclus avec : ``` #include "esp_now.h" ```


Ce fichier d'en-tête fait partie de l'API fournie par le composant esp_wifi. Pour déclarer que votre composant dépend de esp_wifi, ajoutez ce qui suit à votre CMakeLists.txt :
```REQUIRES esp_wifi``` ou ```PRIV_REQUIRES esp_wifi```

## Trouver l'adresse MAC

```
#include "WiFi.h"
 
void setup(){
  Serial.begin(115200);
  WiFi.mode(WIFI_MODE_STA);
  Serial.println(WiFi.macAddress());
}
 
void loop(){

}
```

## Variable pour l'adresse MAC
```
uint8_t broadcastAddress[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
```

## Structure d'envoyer
```
typedef struct struct_message {
  char a[32];
  int b;
  float c;
  bool d;
} struct_message;
```

## Preparation d'une function CallBack d'envoie
```
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
  Serial.print("\r\nLast Packet Send Status:\t");
  Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Delivery Success" : "Delivery Fail");
}
```
## Preparation d'une function CallBack reception
```
void OnDataRecv(const uint8_t * mac, const uint8_t *incomingData, int len) {
  memcpy(&myData, incomingData, sizeof(myData));
  Serial.print("Bytes received: ");
  Serial.println(len);
  Serial.print("Char: ");
  Serial.println(myData.a);
  Serial.print("Int: ");
  Serial.println(myData.b);
  Serial.print("Float: ");
  Serial.println(myData.c);
  Serial.print("Bool: ");
  Serial.println(myData.d);
  Serial.println();
}
```

## Enregistrer le CallBack d'envoie
```
esp_now_register_send_cb(OnDataSent);
```

### Enregistrer le CallBack reception
```
esp_now_register_recv_cb(OnDataRecv);
```

## Setup() Example
```
void setup() {
  // Init Serial Monitor
  Serial.begin(115200);
 
  // Set device as a Wi-Fi Station
  WiFi.mode(WIFI_STA);

  // Init ESP-NOW
  if (esp_now_init() != ESP_OK) {
    Serial.println("Error initializing ESP-NOW");
    return;
  }
```

## Loop() Example d'envoie
```
void loop() {
  // Set values to send
  strcpy(myData.a, "THIS IS A CHAR");
  myData.b = random(1,20);
  myData.c = 1.2;
  myData.d = false;
  
  // Send message via ESP-NOW
  esp_err_t result = esp_now_send(broadcastAddress, (uint8_t *) &myData, sizeof(myData));
   
  if (result == ESP_OK) {
    Serial.println("Sent with success");
  }
  else {
    Serial.println("Error sending the data");
  }
  delay(2000);
}
```

## Loop() Example de reception
```
void loop() {
  ## La reception va tout appeler la function CallBack quand elle recoit un message, donc pas besoin de rien dans la loop
}
```

# Functions

## Init
```esp_err_t esp_now_init(void)```

Initialisez la fonction ESPNOW.

Retour:
<dl>
<dt>ESP_OK : réussir</dt>
<dt>ESP_ERR_ESPNOW_INTERNAL : Erreur interne</dt>
</dl>

## Deinit
```esp_err_t esp_now_deinit(void)```

Désinitialisez la fonction ESPNOW.
Retour
<dl>
<dt>ESP_OK : réussir</dt>
</dl>

## Function de Reception
```esp_err_t esp_now_register_recv_cb(esp_now_recv_cb_t cb)```

Enregistrez la fonction de rappel pour recevoir des données ESPNOW.

Paramètres
cb -- fonction de rappel pour recevoir des données ESPNOW

Retour
<dl>
<dt>ESP_OK : réussir</dt>
<dt>ESP_ERR_ESPNOW_NOT_INIT : ESPNOW n'est pas initialisé</dt>
<dt>ESP_ERR_ESPNOW_INTERNAL : erreur interne</dt>
</dl>

## Function a envoyer
```esp_err_t esp_now_register_send_cb(esp_now_send_cb_t cb)```

Enregistrez la fonction de rappel d'envoi de données ESPNOW.

Paramètres
cb -- fonction de rappel pour l'envoi de données ESPNOW

Retour
<dl>
<dt>ESP_OK : réussir</dt>
<dt>ESP_ERR_ESPNOW_NOT_INIT : ESPNOW n'est pas initialisé</dt>
<dt>ESP_ERR_ESPNOW_INTERNAL : erreur interne</dt>
</dl>

## Envoyer
```esp_err_t esp_now_send(const uint8_t *peer_addr, const uint8_t *data, size_t len)```
Envoyez des données ESPNOW.

***
Attention
1. Si peer_addr n'est pas NULL, envoyez les données au homologue dont l'adresse MAC correspond à peer_addr
2. Si peer_addr est NULL, envoyez les données à tous les homologues ajoutés à la liste d'homologues.
3. La longueur maximale des données doit être inférieure à ESP_NOW_MAX_DATA_LEN
4. Le tampon pointé par l'argument data n'a pas besoin d'être valide après le retour de esp_now_send
***
Paramètres
peer_addr -- adresse MAC homologue

data -- données à envoyer

len -- longueur des données

Retour
<dl>
<dt>ESP_OK : réussir</dt>
<dt>ESP_ERR_ESPNOW_NOT_INIT : ESPNOW n'est pas initialisé</dt>
<dt>ESP_ERR_ESPNOW_ARG : argument invalide</dt>
<dt>ESP_ERR_ESPNOW_INTERNAL : erreur interne</dt>
<dt>ESP_ERR_ESPNOW_NO_MEM : mémoire insuffisante, lorsque cela arrive, vous pouvez attendre un peu avant d'envoyer les données suivantes</dt>
<dt>ESP_ERR_ESPNOW_NOT_FOUND : le correspondant est introuvable</dt>
<dt>ESP_ERR_ESPNOW_IF : l'interface Wi-Fi actuelle ne correspond pas à celle du homologue</dt>
<dt>ESP_ERR_ESPNOW_CHAN : le canal Wi-Fi actuel ne correspond pas à celui du homologue</dt>
</dl>

## Ajout d'un appareil
```esp_err_t esp_now_add_peer(const esp_now_peer_info_t *peer)```

Ajoutez une liste peer to peer.

Paramètres
pair -- informations sur les pairs

Retour
<dl>
<dt>ESP_OK : réussir</dt>
<dt>ESP_ERR_ESPNOW_NOT_INIT : ESPNOW n'est pas initialisé</dt>
<dt>ESP_ERR_ESPNOW_ARG : argument invalide</dt>
<dt>ESP_ERR_ESPNOW_FULL : la liste des pairs est pleine</dt>
<dt>ESP_ERR_ESPNOW_NO_MEM : mémoire insuffisante</dt>
<dt>ESP_ERR_ESPNOW_EXIST : le pair a existé</dt>
</dl>

### Methode contraire

```
esp_err_t esp_now_deinit(void)
esp_err_t esp_now_unregister_recv_cb(void)
esp_err_t esp_now_unregister_send_cb(void)
esp_err_t esp_now_del_peer(const uint8_t *peer_addr)
```
