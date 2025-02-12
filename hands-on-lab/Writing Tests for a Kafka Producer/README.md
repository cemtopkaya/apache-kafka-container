## Açıklama

Birim testi, iyi uygulamalarla yazılım geliştirmenin önemli bir parçasıdır ve bu, özel Kafka üretici kodunuz için bile geçerlidir. Neyse ki Kafka, Kafka üreticileriniz için bu tür testleri kolayca yazmanıza yardımcı olabilecek test fikstürleri sunar. Bu laboratuvarda, bazı mevcut üretici kodları için birkaç birim testi oluşturarak Kafka'nın üretici test fikstürleriyle çalışacağız.

## Hedefler

Aşağıdaki öğrenme hedeflerine ulaşarak bu laboratuvarı başarıyla tamamlayın:

Başlangıç Projesini GitHub'dan Klonlama ve Test Çalışması Gerçekleştirme

1.  Başlangıç projesini GitHub'dan klonlayın:

```shell
cd ~/
git clone https://github.com/linuxacademy/content-ccdak-producer-tests-lab.git
```

2.  Kodun derlenebildiğinden ve çalışabildiğinden emin olmak için bir test çalıştırması gerçekleştirin:

```shell
cd content-ccdak-producer-tests-lab
./gradlew test
```

Kod derlenmelidir, ancak testler henüz uygulanmadıkları için başarısız olmalıdır.

`MemberSignupsProducer` için Birim Testlerini Uygulayın

1.  `MemberSignupsProducer` için test sınıfını düzenleyin:

```bash
vi src/test/java/com/linuxacademy/ccdak/producer/MemberSignupsProducerTest.java
```

2.  `testHandleMemberSignup_sent_data` testini uygulayın:

```java
@Test
public void testHandleMemberSignup_sent_data() {
    // handleMemberSignup çağrıldığında üreticinin doğru verileri doğru konuya gönderdiğini doğrulamak için basit bir test gerçekleştirin.
    // Yayınlanan kaydın anahtar olarak memberId ve değer olarak büyük harfli isme sahip olduğunu doğrulayın.
    // Kayıtların member_signups konusuna gönderildiğini doğrulayın.
    memberSignupsProducer.handleMemberSignup(1, "Summers, Buffy");

    mockProducer.completeNext();

    List<ProducerRecord<Integer, String>> records = mockProducer.history();
    Assert.assertEquals(1, records.size());
    ProducerRecord<Integer, String> record = records.get(0);
    Assert.assertEquals(Integer.valueOf(1), record.key());
    Assert.assertEquals("SUMMERS, BUFFY", record.value());
    Assert.assertEquals("member_signups", record.topic());
}
```

3.  `testHandleMemberSignup_partitioning` testini uygulayın:

```java
@Test
public void testHandleMemberSignup_partitioning() {
    // A-M ile başlayan bir değere sahip kayıtların 0. bölüme, diğerlerinin ise 1. bölüme atandığını doğrulayın.
    // Bu testte iki kayıt gönderebilirsiniz, biri A-M ile başlayan bir değere sahipken diğeri N-Z ile başlayan bir değere sahip olabilir.
    memberSignupsProducer.handleMemberSignup(1, "M");
    memberSignupsProducer.handleMemberSignup(1, "N");

    mockProducer.completeNext();
    mockProducer.completeNext();

    List<ProducerRecord<Integer, String>> records = mockProducer.history();
    Assert.assertEquals(2, records.size());
    ProducerRecord<Integer, String> record1 = records.get(0);
    Assert.assertEquals(Integer.valueOf(0), record1.partition());
    ProducerRecord<Integer, String> record2 = records.get(1);
    Assert.assertEquals(Integer.valueOf(1), record2.partition());
}
```

4.  `testHandleMemberSignup_output` testini uygulayın:

```java
@Test
public void testHandleMemberSignup_output() {
    // Üreticinin kayıt verilerini System.out'a kaydettiğini doğrulayın.
    // System.out verilerini yakalamak için bu sınıfta systemOutContent adlı bir metin fikstürü zaten ayarlanmıştır.
    memberSignupsProducer.handleMemberSignup(1, "Summers, Buffy");

    mockProducer.completeNext();

    Assert.assertEquals("key=1, value=SUMMERS, BUFFY\n", systemOutContent.toString());
}
```

5.  `testHandleMemberSignup_error` testini uygulayın:

```java
@Test
public void testHandleMemberSignup_error() {
    // Bir kayıt eklenirken bir hata oluşursa üreticinin hata mesajını System.err dosyasına kaydettiğini doğrulayın.
    // System.err verilerini yakalamak için bu sınıfta systemErrContent adlı bir metin fikstürü zaten ayarlanmıştır.
    memberSignupsProducer.handleMemberSignup(1, "Summers, Buffy");

    mockProducer.errorNext(new RuntimeException("test hatası"));

    Assert.assertEquals("test hatası\n", systemErrContent.toString());
}
```

6.  Testlerinizi çalıştırın ve geçtiklerinden emin olun:

```shell
./gradlew testi
```
