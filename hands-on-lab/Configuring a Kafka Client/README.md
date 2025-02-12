## Açıklama

Üreticiler ve tüketiciler gibi Kafka istemcileri, brokerlar ve konular gibi yapılandırılabilir. Bu uygulamalı laboratuvarda, Java'da yazılmış basit bir üreticide bazı yapılandırma değişiklikleri yaparak Kafka istemcilerini yapılandırmanın temellerini keşfetme fırsatına sahip olacaksınız. Bu laboratuvarı tamamladıktan sonra, Kafka istemcilerini programatik olarak yapılandırma süreciyle ilgili biraz deneyim kazanacaksınız.

## Ek Kaynaklar

Süpermarket şirketinizin Java ile yazılmış bir Kafka üretici uygulaması var. Sizden bu uygulamada bazı yapılandırma değişiklikleri yapmanız ve ardından bu değişiklikleri test etmek için uygulamayı çalıştırmanız istendi. Temel uygulama kodu zaten yazılmıştır. Değiştirmek ve test etmek için GitHub'dan kaynak kodun bir kopyasını klonlayabilirsiniz. Gerekli yapılandırma değişikliklerini uygulayın ve her şeyin beklendiği gibi çalıştığını doğrulamak için programı çalıştırın.

Bunlar, üretici için uygulamanız gereken yapılandırma değişiklikleridir:

- Bir bölüm liderinin başarısız olması durumunda maksimum veri bütünlüğünü sağlamak için `acks` değerini `all` olarak ayarlayın.
- Üreticinin mesajları tamponlaması için daha az miktarda bellek ayrılması gerekir. Buffer.memory` değerini `12582912` olarak ayarlayın (yaklaşık 12 MB).
- Üreticinin boşta kalan bağlantıları varsayılan ayarın belirttiğinden daha hızlı bir şekilde temizlemesi gerekecektir. connections.max.idle.ms` değerini `300000` ms (5 dakika) olarak ayarlayın.

Üretici proje kodunu [https://github.com/linuxacademy/content-ccdak-kafka-client-config-lab.git](https://github.com/linuxacademy/content-ccdak-kafka-client-config-lab.git) adresinde bulabilirsiniz. Bu projeyi `Broker 1` üzerinde `/home/cloud_user` içine klonlayın. Üretici, `src/main/java/com/linuxacademy/ccdak/client/config/Main.java` adresinde bulunan `Main` sınıfında uygulanmaktadır.

Değişikliklerinizi test etmek için `/home/cloud_user/content-ccdak-kafka-client-config-lab` dizinindeyken bu komutu çalıştırarak üreticiyi çalıştırabilirsiniz:

```shell
./gradlew çalıştır
```

Yayıncı tarafından konuya yayınlanan çıktı verilerini görüntülemek istiyorsanız, bu komutu kullanın:

```shell
kafka-console-consumer --bootstrap-server localhost:9092 --topic inventory_purchases --property print.key=true --from-beginning
```

## Hedefler

Aşağıdaki öğrenme hedeflerine ulaşarak bu laboratuvarı başarıyla tamamlayın:

Üretici Kaynak Kodunu Klonlayın ve Her Şeyin Çalıştığından Emin Olmak İçin Çalıştırın

1.  Üretici kaynak kodunu ev dizinine klonlayın:

```shell
cd ~/
git clone https://github.com/linuxacademy/content-ccdak-kafka-client-config-lab.git
```

1.  Kodu değiştirmeden önce çalıştığından emin olmak için çalıştırın:

```shell
cd content-ccdak-kafka-client-config-lab
./gradlew çalıştır
```

1.  Çıktıyı görüntülemek için `inventory_purchases` başlığındaki kayıtları kullanın:

```shell
kafka-console-consumer --bootstrap-server localhost:9092 --topic inventory_purchases --property print.key=true --from-beginning
```

Yapımcıda Gerekli Yapılandırma Değişikliklerini Uygulayın ve Test Etmek için Programı Çalıştırın

1.  Üretici kaynak kodunun `Main` sınıfını düzenleyin:

```shell
vi src/main/java/com/linuxacademy/ccdak/client/config/Main.java
```

1.  Üretici örneklenmeden önce gerekli konfigürasyonları `props` nesnesine ekleyin. Son kod aşağıdaki gibi görünmelidir:

```java
paket com.linuxacademy.ccdak.client.config;

import java.util.Properties;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class Main {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");


        // Üreticinin bir isteği tamamlanmış olarak kabul etmeden önce liderin alması gereken onay sayısını ayarlayın.
        // "all" değeri, liderin kaydı onaylamak için tüm senkronize replikaların onayını bekleyeceği anlamına gelir.
        // Üreticinin sunucuya gönderilmeyi bekleyen kayıtları tamponlamak için kullanabileceği toplam bellek baytını ayarlayın.
        // Bir bağlantının kapanmadan önce açık kalabileceği maksimum boşta kalma süresini ayarlayın.
        props.put("acks", "all");
        props.put("buffer.memory", "12582912");
        props.put("connections.max.idle.ms", "300000");

        Producer<String, String> producer = new KafkaProducer<>(props);
        producer.send(new ProducerRecord<>("inventory_purchases", "apples", "1"));
        producer.send(new ProducerRecord<>("inventory_purchases", "apples", "3"));
        producer.send(new ProducerRecord<>("inventory_purchases", "oranges", "12"));
        producer.send(new ProducerRecord<>("inventory_purchases", "bananas", "25"));
        producer.send(new ProducerRecord<>("inventory_purchases", "pears", "15"));
        producer.send(new ProducerRecord<>("inventory_purchases", "apples", "6"));
        producer.send(new ProducerRecord<>("inventory_purchases", "pears", "7"));
        producer.send(new ProducerRecord<>("inventory_purchases", "oranges", "1"));
        producer.send(new ProducerRecord<>("inventory_purchases", "grapes", "56"));
        producer.send(new ProducerRecord<>("inventory_purchases", "oranges", "11"));
        producer.close();
    }

}
```

1.  Programı çalıştırın:

```shell
./gradlew çalıştır
```

1.  Üretici tarafından oluşturulan yeni kayıtları görebildiğimizi doğrulamak için `inventory_purchases` başlığındaki kayıtları tüketin:

```shell
kafka-console-consumer --bootstrap-server localhost:9092 --topic inventory_purchases --property print.key=true --from-beginning
```
