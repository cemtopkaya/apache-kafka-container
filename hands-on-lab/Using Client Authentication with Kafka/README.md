## Açıklama
Kafka broker'larınızı dış istemcilerin onlarla iletişim kurabileceği şekilde açığa çıkarmak istiyorsanız, uygulamanızın güvenli olduğundan emin olmanız önemlidir. Bazı durumlarda bu, istemci kimlik doğrulamasını uygulamak anlamına gelir. Bu laboratuvarda, Kafka'da mevcut istemci kimlik doğrulama yöntemlerinden biriyle uygulamalı olarak çalışma fırsatına sahip olacaksınız: istemci sertifikaları. Mevcut bir kümede istemci sertifikalarıyla kimlik doğrulamasını uygulayacak ve ardından uygulamanızın çalıştığını doğrulamak için istemci olarak kimlik doğrulaması yapacaksınız. Bu, size bir Kafka kümesini güvence altına alma süreciyle ilgili uygulamalı deneyim kazandıracaktır.

## Hedefler

Aşağıdaki öğrenme hedeflerine ulaşarak bu laboratuvarı başarıyla tamamlayın:

İstemci Sertifika Dosyalarınızı Oluşturma

1.  Bir istemci sertifikası oluşturun. İstendiğinde istemci anahtar deposu için bir parola seçin:

```shell
cd ~/certs/

keytool -keystore client.keystore.jks -alias kafkauser -validity 365 -genkey -keyalg RSA -dname “CN=kafkauser, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown"
```

1.  Anahtarı imzalayın, ardından sertifika yetkilisini ve imzalı anahtarı `keystore`a aktarın. ca-key` şifresi sorulduğunda, `AllTheKeys` şifresini girin:

```shell
keytool -keystore client.keystore.jks -alias kafkauser -certreq -file client-cert-file

openssl x509 -req -CA ca-cert -CAkey ca-key -in client-cert-file -out client-cert-signed -days 365 -CAcreateserial

keytool -keystore client.keystore.jks -alias CARoot -import -file ca-cert

keytool -keystore client.keystore.jks -alias kafkauser -import -file client-cert-signed
```

1.  İstemci anahtar deposunu uygun bir konuma taşıyın:

```shell
sudo cp client.keystore.jks /var/private/ssl/

sudo chown root:root /var/private/ssl/client.keystore.jks
```

Broker için İstemci Kimlik Doğrulamasını Etkinleştir

1.  İstemci kimlik doğrulamasını `server.properties` içinde `required` olarak ayarlayın:

```shell
sudo vi /etc/kafka/server.properties
```

1.  ssl.client.auth` ile başlayan satırı bulun ve değiştirin:

```ini
ssl.client.auth=required
```

1.  Kafka'yı yeniden başlatın ve ardından her şeyin çalıştığını doğrulayın:

```shell
sudo systemctl restart confluent-kafka

sudo systemctl status confluent-kafka
```

İstemci Kimlik Doğrulama Ayarlarını İstemci Yapılandırma Dosyanıza Ekleme

1.  client-ssl.properties` dosyasını düzenleyin:

```shell
cd ~/

vi client-ssl.properties
```

1.  Aşağıdaki satırları ekleyin:

```ini
ssl.keystore.location=/var/private/ssl/client.keystore.jks
ssl.keystore.password=<istemci anahtar deposu parolanız>
ssl.key.password=<istemci anahtar parolanız>
```

1.  Her şeyin çalıştığını doğrulamak için istemci kimlik doğrulamasını kullanarak bir konsol tüketicisi oluşturun:

```shell
kafka-console-consumer --bootstrap-server zoo1:9093 --topic inventory_purchases --from-beginning --consumer.config client-ssl.properties
```
