---
title: "FastAPI Uygulamasına Çoklu Dil Desteği Ekleme"
tags: ["fastapi"]
categories: ["FastAPI"]
date: 2023-07-30T19:34:18+03:00
draft: false
slug: "fastapi-setting-gettext-translations-tr"
featured_image: "https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png"
---

Merhaba!

Bu blog yazısında, FastAPI uygulamanızda dil çevirilerini Gettext kullanarak nasıl yapılandıracağınızı öğreneceksiniz. Gettext, dil desteği eklemek için oldukça etkili bir araçtır ve modern web uygulamaları için harika bir seçenektir.

Aşağıdaki gibi bir uygulamamız olduğunu varsayalım ve buna çeviri desteği eklemeyi adım adım oluşturalım.
```python
# main.py

from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    message = "Merhaba, Dünya!"
    return {"message": message}
```


## Adım 1: Paket kurulumu

İlk adım olarak, Gettext'i FastAPI uygulamanıza eklemek için `babel` kütüphanesini yüklemeniz gerekecek. Gereksinimleri karşılamak için aşağıdaki komutu kullanabilirsiniz:

```bash
pip install babel
```

## Adım 2: Çevirilecek Metinleri İşaretleyin

Çeviri yapmak istediğiniz metinleri işaretlemek için gettext fonksiyonunu kullanacağız. Bu değişken, metinleri çeviri için hazırlayacak. Örneğin, FastAPI uygulamanızda şu şekilde kullanabilirsiniz:

```python
# main.py

from fastapi import FastAPI
from gettext import gettext as _

app = FastAPI()

@app.get("/")
async def read_root():
    message = _("Merhaba, Dünya!")
    return {"message": message}
```

## Adım 3: Çeviri Dosyalarını Oluşturun

Metinleri işaretledikten sonra, çeviri dosyalarınızı oluşturmanız gerekecek. Bu dosyalar, çeviri yapılan metinleri belirli dillere göre saklayacaktır. Çeviri dosyalarınızı oluşturmak için babel paketini kullanabilirsiniz. Aşağıdaki komutları çalıştırarak çeviri dosyalarınızı oluşturabilirsiniz:

```bash
# daha düzenli bir yapı için tüm dil dosyalarını bir klasörde tutalım
mkdir languages

# Örnek olarak main.py dosyasındaki çevirileri extract edelim
pybabel extract -o languages/messages.pot main.py

# Uygulamanın destekleyeceği dillerin çeviri dosyalarını init edelim
pybabel init -l en_US -i languages/messages.pot -d languages
pybabel init -l tr_TR -i languages/messages.pot -d languages 
```

## Adım 4: Çevirileri Yapın
.po uzantılı dosyalar, çevirilerinizi belirli bir dile çevirmenize olanak tanır. Çeviri işlemi tamamlandıktan sonra, bu dosyalar .mo uzantısına dönüştürülecektir. Çeviri dosyalarınızı düzenlemek ve çevirileri yapmak için bir metin düzenleyici veya çeviri aracı kullanabilirsiniz.

Örnek olarak `languages/en_US/LC_MESSAGES/messages.po` aşağıdaki gibi değiştirelim:
```
#: main.py:8
msgid "Merhaba, Dünya!"
msgstr "Hello, World!"
```
 
Çevirileri tamamladıktan sonra compile edilmesi gerekiyor, aşağıdaki komut ile compile edelim:
```bash
pybabel compile -d languages
```

## Adım 5: Çevirileri Uygulamaya Ekleyin
Çeviri dosyalarınızı tamamladıktan sonra, uygulamanızda çeviri desteğini etkinleştirebilirsiniz. Bunu yapmak için aşağıdaki kodu uygulamanıza ekleyebilirsiniz:

```python
# main.py

from fastapi import FastAPI
from gettext import translation

app = FastAPI()

# Çeviri dosyalarınızın yolu ve dili belirtin
trans = translation('messages', 'languages', languages=['en_US'])
trans.install()


@app.get("/")
async def read_root():
    message = _("Merhaba, Dünya!")
    return {"message": message}
```

Artık uygulamamız en_US dosyasındaki çeviri ile response dönüyor. Fakat şuandaki yapı kullanıcıdan alınan veriye göre dinamik bir response dönmüyor.

## Adım 6: Dil ayarını değiştirilebilir yapmak
Kullanıcı diline göre response'u değiştirmek için öncelikle kullanıcıdan bir dil ayarı alalım. Bu ayarı alacağınız yeri ihtiyacınıza göre değiştirebilirsiniz, biz örneğin basit kalması açısından Header'dan alalım. Bunun için bir Middleware yazalım.

```python
# main.py

from fastapi import FastAPI, Request
from gettext import translation

app = FastAPI()

default_language = "tr_TR"
languages = {
    "en": "en_US",
    "tr": "tr_TR",
}

def get_language(request: Request):
    user_language = request.headers.get('accept-language', 'en')
    return languages.get(user_language, default_language)

@app.middleware("http")
async def add_gettext_middleware(request: Request, call_next):
    language = get_language(request)
    trans = translation('messages', 'languages', languages=[language])
    trans.install()
    response = await call_next(request)
    return response


@app.get("/")
async def read_root():
    message = _("Merhaba, Dünya!")
    return {"message": message}
```

Artık FastAPI uygulamanız kullanıcının header'da gönderdiği dil bilgisine göre dinamik çeviri yapacak ve kullanıcılara tercih ettikleri dillerde içerik sunacaktır. Test etmek için alttaki şekilde curl isteği atabilirsiniz:
```bash
# İngilizce response için
curl -X 'GET' \
  'http://127.0.0.1:8000' \
  -H 'accept-language: en'

# Türkçe response için
curl -X 'GET' \
  'http://127.0.0.1:8000' \
  -H 'accept-language: tr'
```


## Sonuç
Bu yazıda, FastAPI uygulamanızda Gettext ile çeviri ayarlarını nasıl yapılandıracağınızı öğrendiniz. Gettext sayesinde uygulamanızı daha fazla kullanıcı için daha erişilebilir hale getirebilir ve çok dilli içerik sunabilirsiniz. FastAPI ve Gettext kombinasyonu, uygulamanızı daha geniş bir kitleye ulaştırmak için güçlü bir araçtır.



Görüşmek üzere!