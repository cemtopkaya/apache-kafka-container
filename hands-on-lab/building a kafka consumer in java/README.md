
## Ek Kaynaklar

Süpermarket şirketiniz envanterle ilgili verileri yönetmek için Kafka kullanıyor. İşlemlerle ilgili verileri içeren bazı dosyaları var ve sizden bu dosyaları okuyabilen ve verileri Kafka'da yayınlayabilen bir üretici oluşturmanızı istiyorlar.

Bu verilerden bazılarının bir örneğini içeren örnek bir işlem günlüğü dosyası var. Dosya `<product>:<quantity>` biçiminde veriler içerir, örneğin: `apples:5`. Dosyadaki her satır yeni bir işlemi temsil eder. Dosyadaki her satırı okuyan ve `inventory_purchases` konusuna bir kayıt yayınlayan bir üretici oluşturun. Anahtar olarak ürün adını ve değer olarak miktarı kullanın.

Şirket ayrıca `elma` alımlarını ayrı bir konuda (`inventory_purchases` konusuna ek olarak) izlemek istiyor. Bu nedenle, anahtarı `elma` olan kayıtları hem `inventory_purchases` hem de `apple_purchases` konularında yayınlayın.

Son olarak, maksimum veri bütünlüğünü korumak için üreticiniz için `acks` değerini `all` olarak ayarlayın.

GitHub'da üreticinizi uygulamak için kullanabileceğiniz bir başlangıç projesi bulunmaktadır: [https://github.com/linuxacademy/content-ccdak-kafka-producer-lab.git](https://github.com/linuxacademy/content-ccdak-kafka-producer-lab.git). Bu projeyi klonlayın ve üreticiyi Main sınıfında uygulayın. Ana sınıfı proje dizininden `./gradlew run` komutu ile çalıştırabilirsiniz.

Örnek işlem günlüğü dosyası başlangıç projesinin içinde `src/main/resources/sample_transaction_log.txt` adresinde bulunabilir.

```text
~/content-ccdak-kafka-producer-lab$ cat src/main/resources/sample_transaction_log.txt 
apples:21
pears:5
beets:7
bananas:8
radishes:34
oranges:7
plantains:27
apples:6
grapes:105
bananas:6
apples:3
radishes:11
pears:9
onions:6
apples:1
plums:8
oranges:1
grapes:206
bananas:3
```

## Hedefler

Aşağıdaki öğrenme hedeflerine ulaşarak bu laboratuvarı başarıyla tamamlayın:

Başlangıç Projesini Klonlayın ve Çalıştığını Doğrulamak için Çalıştırın

1.  Başlangıç projesini `home` dizinine klonlayın:

```shell
cd ~/
git clone https://github.com/linuxacademy/content-ccdak-kafka-producer-lab.git
```

1.  Kodu değiştirmeden önce çalıştığından emin olmak için çalıştırın:

```shell
cd content-ccdak-kafka-producer-lab/
./gradlew çalıştır
```

**Not:** Çıktıda bir `Hello, world!` mesajı görmeliyiz.

Üreticiyi Uygulayın ve Çalıştığını Doğrulamak için Çalıştırın

1.  Ana sınıfı düzenleyin:

```shell
nano src/main/java/com/linuxacademy/ccdak/producer/Main.java
```

1.  Üreticiyi sağlanan spesifikasyona göre uygulayın:

```java
  paket com.linuxacademy.ccdak.producer;
  
  // Dosya okuma işleri için
  import java.io.BufferedReader;
  import java.io.File;
  import java.io.FileReader;
  import java.io.IOException;
  
  // Kafka bağlntısı için
  import java.util.Properties;
  import org.apache.kafka.clients.producer.KafkaProducer;
  import org.apache.kafka.clients.producer.Producer;
  import org.apache.kafka.clients.producer.ProducerRecord;

  public class Main {

      public static void main(String[] args) {
          // Kafka bağlantısının özelliklerini vereceğiz
          Properties props = new Properties();

          // Yerel bilgisayarda yer alan Kafka'ya bağlanacağız
          props.put("bootstrap.servers", "localhost:9092");
          
          // Veriyi serileştirme ve deserileştirme kafkaya bağlantı kuran üretici/tüketicinin görevi
          props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
          props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

          // Bu ayar, tüm replikaların mesajı aldığını onaylayana kadar üreticinin bir onay beklemesini sağlar.
          props.put("acks", "all");

          Producer<String, String> producer = new KafkaProducer<>(props);

          try {
              // Dosya sonuna kadar okuyup satırları kafkaya gönderecek
              File file = new File(Main.class.getClassLoader().getResource("sample_transaction_log.txt").getFile());
              BufferedReader br = new BufferedReader(new FileReader(dosya));
              String satır;
              while ((line = br.readLine()) != null) {
                  String[] lineArray = line.split(":");
                  String key = lineArray[0];
                  String value = lineArray[1];
                  producer.send(new ProducerRecord<>("inventory_purchases", key, value));
                  // Eğer satırda okuduğumuz anahtar:değer ikilisinde anahtar apples ise apple_purchases topic'e gitsin
                  if (key.equals("apples")) {
                      producer.send(new ProducerRecord<>("apple_purchases", key, value));
                  }
              }
              br.close();
          } catch (IOException e) {
              // IOException'u sarıp fırlatıyoruz çünkü checked exception ve burada handle etmek istemiyoruz
              throw new RuntimeException(e);
          }

          producer.close();
      }

  }
```

1.  Programı çalıştırın:

```
./gradlew çalıştır
```

1.  Kayıtları `inventory_purchases` başlığından tüketin ve üretici tarafından oluşturulan yeni kayıtları görebildiğimizi doğrulayın:

```
 kafka-console-consumer --bootstrap-server localhost:9092 --topic inventory_purchases --property print.key=true --from-beginning
```

1.  Üretici tarafından oluşturulan yeni kayıtları görebildiğimizi doğrulamak için `apple_purchases` başlığındaki kayıtları tüketin:

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic apple_purchases --property print.key=true --from-beginning
```


Translated with DeepL.com (free version)