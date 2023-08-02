##Django OAuth Toolkit
http://127.0.0.1:8000/o/applications/1/
web-app
client_id=client_id
client_secret=client_secret
client_type=confidential
grant_type=authorization-code
Redirect Uris=http://127.0.0.1:8000/noexist/callback


export ID=client_id
export SECRET=client_secret

python code_challenge.py
export CODE_VERIFIER=WjAwV1Y3MERBRUdZUkMxMUZYSjMwVU5XNFZSNlFQSTIzNDJOSDNVSktET0s1VlcyWlM2NjYyTVZCTzlFWU5IQkRSV1NXTlRWVEsxMTBQOE9NRVU3MEZCUTM1SktUTlNFMkRSVFA2MQ==
code_challenge=iN7vuqB05lLR4wGOlVyOwBDuSmvhO32tAqN9NqTTlQ4


http://127.0.0.1:8000/o/authorize/?response_type=code&code_challenge=iN7vuqB05lLR4wGOlVyOwBDuSmvhO32tAqN9NqTTlQ4&code_challenge_method=S256&client_id=client_id&redirect_uri=http://127.0.0.1:8000/noexist/callback
http://127.0.0.1:8000/noexist/callback?code=ND8aZBnaVvMyDfXUa8ATaRPLMF8uMD
export CODE=ND8aZBnaVvMyDfXUa8ATaRPLMF8uMD

curl -X POST \
    -H "Authorization: Basic ${CREDENTIAL}" \
    -H "Cache-Control: no-cache" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    "http://127.0.0.1:8000/o/token/" \
    -d "grant_type=client_credentials"



##Тонкости авторизации: обзор технологии OAuth 2.0
https://oauth.net/2/grant-types/authorization-code/

https://habr.com/ru/companies/flant/articles/475942/

https://habr.com/ru/companies/dododev/articles/520046/

https://habr.com/ru/articles/535174/



#Роли

В OAuth 2.0 термины:

Resource owner - сущность, которая имеет права доступа на защищённый ресурс. Сущность может быть конечным пользователем или какой-либо системой. Защищённый ресурс — это HTTP endpoint, которым может быть что угодно: API endpoint, файл на CDN, web-сервис.

Client - Приложение (например, сервис Terrible Pun of the Day), которое хочет получить доступ или выполнить определенные действия от имени Resource Owner'а.

Authorization server - Приложение, которое знает Resource Owner'а и в котором у Resource Owner'а уже есть учетная запись.

Resource server - Программный интерфейс приложения (API) или сервис, которым Client хочет воспользоваться от имени Resource Owner'а.

Redirect URI - Ссылка, по которой Authorization Server перенаправит Resource Owner'а после предоставления разрешения Client'у. Иногда ее называют «Возвратным URL» («Callback URL»).

Response Type - Тип информации, которую ожидает получить Client. Самым распространенным Response Type'ом является код, то есть Client рассчитывает получить Authorization Code.

Scope - Это подробное описание разрешений, которые требуются Client'у, такие как доступ к данным или выполнение определенных действий.

Consent - Authorization Server берет Scopes, запрашиваемые Client'ом, и спрашивает у Resource Owner'а, готов ли тот предоставить Client'у соответствующие разрешения.

Client ID - Этот ID используется для идентификации Client'а на Authorization Server'е.

Client Secret - Это пароль, который известен только Client'у и Authorization Server'у. Он позволяет им конфиденциально обмениваться информацией.

Authorization Code - Временный код с небольшим периодом действия, который Client предоставляет Authorization Server'у в обмен на Access Token.

Access Token - Ключ, который клиент будет использовать для связи с Resource Server'ом. Этакий бейдж или ключ-карта, предоставляющий Client'у разрешения на запрос данных или выполнение действий на Resource Server'е от вашего имени.


#При регистрации, кроме ID клиента, должны быть обязательно указаны 2 параметра: redirection URI и client type.

Redirection URI — адрес, на который отправится владелец ресурса после успешной авторизации. Кроме авторизации, адрес используется для подтверждения, что сервис, который обратился за авторизацией, тот, за кого себя выдаёт.

Client type — тип клиента, от которого зависит способ взаимодействия с ним. Тип клиента определяется его возможностью безопасно хранить свои учётные данные для авторизации — токен. Поэтому существует всего 2 типа клиентов:

Confidential — клиент, который может безопасно хранить свои учётные данные. Например, к такому типу клиентов относят web-приложения, имеющие backend.

Public — не может безопасно хранить свои учётные данные. Этот клиент работает на устройстве владельца ресурса, например, это браузерные или мобильные приложения.

##Grant
Grant — это данные, которые представляют из себя успешную авторизацию клиента владельцем ресурса, используемые клиентом для получения access token.

Например, когда мы где-либо аутентифицируемся с помощью Google, перед глазами всплывает уведомление. В нём говорится, что такой-то сервис хочет получить доступ к данным о вас или к вашим ресурсам (выводятся запрашиваемые scope-token). Это уведомление называется «Consent Screen».

В момент, когда нажимаем «ОК», в базу данных попадает тот самый grant: записываются данные о том, что такой-то пользователь дал такие-то доступы такому-то сервису. Клиент получает какой-то идентификатор успешной аутентификации, например строку, которая ассоциируется с данными в базе данных.

Существует 4 + 1 способа получения grant — grant type:

Authorization code — используется для confedencial клиентов — web-сервисов.

Client credentials — используется для confedential клиентов, которые запрашивают доступ к своим ресурсам или ресурсам, заранее согласованным с сервером авторизации.

Implicit — использовался public-клиентами, которые умеют работать с redirection URI (например, для браузерных и мобильных приложений), но был вытеснен authorization code grant с PKCE (Proof Key for Code Exchange — дополнительная проверка, позволяющая убедиться, что token получит тот же сервис, что его и запрашивал. Прочитать подробнее — RFC 7636).

Resource owner password credentials. В RFC 6819, посвящённому безопасности в OAuth 2.0, данный тип grant считается ненадёжным. Если раньше его  разрешалось использовать только для миграции сервисов на OAuth 2.0, то в данный момент его не разрешено использовать совсем.

Device authorization (добавлен в RFC 8628) – используется для авторизации устройств, которые могут не иметь веб-браузеров, но могут работать через интернет. Например, это консольные приложения, умные устройства или Smart TV.

Актуальными можно считать только authorization code (с PKCE), client credentials и device authorization grant, но мы рассмотрим все. Рассматривать grant будем в порядке возрастания сложности понимания.


##Authorization code
OAuth flow:

1 Вы, Resource Owner, желаете предоставить сервису Terrible Pun of the Day (Client'у) доступ к своим контактам, чтобы тот мог разослать приглашения всем вашим друзьям.

2 Client перенаправляет браузер на страницу Authorization Server'а и включает в запрос Client ID, Redirect URI, Response Type и одно или несколько Scopes (разрешений), которые ему необходимы.

3 Authorization Server проверяет вас, при необходимости запрашивая логин и пароль.

4 Authorization Server выводит форму Consent (подтверждения) с перечнем всех Scopes, запрашиваемых Client'ом. Вы соглашаетесь или отказываетесь.

5 Authorization Server перенаправляет вас на сайт Client'а, используя Redirect URI вместе с Authorization Code (кодом авторизации).

6 Client напрямую связывается с Authorization Server'ом (в обход браузера Resource Owner'а) и безопасно отправляет Client ID, Client Secret и Authorization Code.

7 Authorization Server проверяет данные и отвечает с Access Token'ом (токеном доступа).

8 Теперь Client может использовать Access Token для отправки запроса на Resource Server с целью получить список контактов.

Задолго до того, как вы разрешили Terrible Pun of the Day получить доступ к контактам, Client и Authorization Server установили рабочие отношения. Authorization Server сгенерировал Client ID и Client Secret (иногда их называют App ID и App Secret) и отправил их Client'у для дальнейшего взаимодействия в рамках OAuth.

Название намекает, что Client Secret должен держаться в тайне, чтобы его знали только Client и Authorization Server. Ведь именно с его помощь Authorization Server подтверждает истинность Client'а

OAuth 2.0 разработан только для авторизации — для предоставления доступа к данным и функциям от одного приложения другому. OpenID Connect (OIDC) — это тонкий слой поверх OAuth 2.0, добавляющий сведения о логине и профиле пользователя, который вошел в учетную запись. Организацию логин-сессии часто называют аутентификацией [authentication], а информацию о пользователе, вошедшем в систему (т.е. о Resource Owner'е), — личными данными [identity]. Если Authorization Server поддерживает OIDC, его иногда называют поставщиком личных данных [identity provider], поскольку он предоставляет Client'у информацию о Resource Owner'е.

OpenID Connect позволяет реализовывать сценарии, когда единственный логин можно использовать во множестве приложений, — этот подход также известен как single sign-on (SSO). Например, приложение может поддерживать SSO-интеграцию с социальными сетями, такими как Facebook или Twitter, позволяя пользователям использовать учётную запись, которая у них уже имеется и которую они предпочитают использовать.

Поток (flow) OpenID Connect выглядит так же, как и в случае OAuth. Единственная разница в том, что в первичном запросе используемый конкретный scope — openid, — а Client в итоге получает как Access Token, так и ID Token.
