**<h1 dir="rtl">authin-doc</h1>**
<h2 dir="rtl">راهنمای اتصال از طریق پروتکل‌های OAuth 2.0 و OpenID Connect</h2>

<h2 dir="rtl">Authorization Code Flow:</h2>
<p dir="rtl">این روش احراز هویت یکی از امن‌ترین و رایج‌ترین روش‌های موجود در پروتکل‌های OpenID Connect و OAuth 2.0 است که برای استفاده توسط سامانه‌های سمت سرور طراحی شده است. فرایند احراز هویت در این روش از دو مرحله دریافت کد موقت (<code>Authorization Code</code>) در تعامل با کاربر و دریافت توکن توسط سرور تشکیل شده است.</p>

<h3 dir="rtl">مرحله اول:</h3>
<p dir="rtl">در این مرحله سامانه RP (سامانه سرویس‌گیرنده) کاربر را جهت احراز هویت و دریافت <code>Authorization Code</code> به آدرس <code>Authorization Endpoint</code> به همراه پارامترهای زیر هدایت می‌کند.</p>
<ol dir="rtl">
	<li><code>client_id:</code> شناسه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>redirect_uri:</code> آدرسی که پس از احراز هویت، کاربر به همراه <code>Authorization Code</code> به این آدرس بازگردانده می‌شود.</li>
	<li><code>scope:</code> این پارامتر مجموعه‌ای از رشته‌های جدا شده با کاراکتر فاصله (space)  و در فرمت <code>UrlEncoded</code> است. جهت استفاده از پروتکل OpenID Connect لازم است مقدار <code>openid</code> در میان این مقادیر وجود داشته باشد. همچنین جهت درخواست دسترسی به اطلاعات پایه پروفایل کاربر از <code>UserInfo Endpoint</code> می‌توان مقدار <code>profile</code> را نیز به آن افزود. در صورتی که مدیریت دسترسی سامانه RP توسط سامانه احراز هویت آتین انجام شود می‌توان لیست دسترسی‌های مورد نظر را نیز به این پارامتر افزود. همچنین در صورتی که هیچ دسترسی‌ای در این پارامتر درخواست نشود، تمامی دسترسی‌های کاربر در توکن ارائه می‌شود.</li>
	<li><code>response_type:</code>  مقدار این پارامتر در این روش باید برابر با <code>code</code> قرار داده شود.</li>
	<li><code>state(اختیاری):</code> این رشته توسط سامانه احراز هویت پردازش نشده و عینا به همین شکل در پاسخ به سامانه RP بازگردانده می‌شود.</li>
  <li><code>claims(اختیاری):</code> در این قسمت می‌توانید برای خصیصه‌های پروفایل کاربر که به سامانه RP ارائه شده است درخواست دهید. این پارامتر به صورت <code>UrlEncoded</code> است.</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```
https://<authin_idp_address>/openidauthorize?client_id=YOUR_CLIENT_ID&redirect_uri=https://YOUR_APP/callback&scope=openid profile&response_type=code&state=YOUR_STATE&claims={"id_token":{"YOUR_REQUESTED_CLAIM1":null,"YOUR_REQUESTED_CLAIM2":null}}
```

**<p dir="rtl">نمونه پاسخ:</p>**

```
https://YOUR_APP/callback?code=YOUR_AUTHORIZATION_CODE&state=YOUR_STATE
```

<h3 dir="rtl">مرحله دوم:</h3>
<p dir="rtl">در این مرحله سامانه RP مقدار <code>Authorization Code</code> دریافتی در مرحله اول را به سمت سرور خود ارسال کرده و مستقیما از طریق سرور خود اقدام به ارسال درخواست توکن با استفاده از مکانیزم <code>POST</code> و به صورت <code>x-www-form-urlencoded</code> به <code>Token Endpoint</code> سامانه احراز هویت آتین می‌نماید. پارامترهای لازم در این مرحله عبارتند از:</p>
<ol dir="rtl">
	<li><code>client_id:</code> شناسه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>client_secret:</code> گذرواژه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>grant_type:</code> این مقدار باید برابر با <code>authorization_code</code> باشد.</li>
	<li><code>code:</code> مقدار <code>Authorization Code</code> دریافتی در مرحله اول</li>
	<li><code>redirect_uri:</code> این مقدار باید عینا با مقدار ارائه شده در مرحله اول یکسان باشد.</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```bash
curl --request POST \
  --url 'https://<authin_idp_address>/api/v1/oauth/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data client_id=YOUR_CLIENT_ID \
  --data client_secret=YOUR_CLIENT_SECRET \
  --data grant_type=authorization_code \
  --data code=YOUR_AUTHORIZATION_CODE \
  --data redirect_uri=https://YOUR_APP/callback
```

**<p dir="rtl">نمونه پاسخ:</p>**

```json
{
    "id_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxIn0.rNjxxxxxxxxxxbbc",
    "access_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxCJ9.glhxxxxxxxxxxzlI",
    "refresh_token": "fwqxxxxxxxxxxK9s",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

<h3 dir="rtl">به روز رسانی توکن (Refresh Token):</h3>
<p dir="rtl">پس  از اتمام فرایند احراز هویت یکی از توکن‌های دریافتی <code>Refresh Token</code> است که سامانه RP با ارائه این توکن به سامانه احراز هویت آتین می‌تواند بدون مداخله کاربر توکن جدیدی دریافت کند. مکانیزم درخواست <code>Refresh Token</code> بدین صورت است که در صورتی که عمر توکن فعلی کاربر به اتمام رسیده و توکن منقضی شده باشد،  سامانه RP با ارائه <code>Refresh Token</code> دریافتی، توکن جدیدی را به دست می‌آورد. لازم به ذکر است که به هنگام اجرای قرایند خروج از سامانه RP، باید <code>Refresh Token</code>  پاک شود.</p>
<p dir="rtl">سامانه RP جهت دریافت توکن جدید باید درخواست <code>Refresh Token</code> را با استفاده از مکانیزم <code>POST</code> و به صورت <code>x-www-form-urlencoded</code> به <code>Token Endpoint</code> سامانه احراز هویت آتین ارائه دهد. پارامترهای این درخواست عبارتند  از:</p>
<ol dir="rtl">
	<li><code>client_id:</code> شناسه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>client_secret:</code> گذرواژه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>grant_type:</code> این مقدار باید برابر با <code>refresh_token</code> باشد.</li>
	<li><code>refresh_token:</code> مقدار <code>Refresh Token</code> دریافتی در هنگام احراز هویت کاربر</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```bash
curl --request POST \
  --url 'https://<authin_idp_address>/api/v1/oauth/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data client_id=YOUR_CLIENT_ID \
  --data client_secret=YOUR_CLIENT_SECRET \
  --data grant_type=refresh_token \
  --data refresh_token=YOUR_REFRESH_TOKEN
```

**<p dir="rtl">نمونه پاسخ:</p>**

```json
{
    "id_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxIn0.rNjxxxxxxxxxxbbc",
    "access_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxCJ9.glhxxxxxxxxxxzlI",
    "refresh_token": "fwqxxxxxxxxxxK9s",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

<h2 dir="rtl">Resource Owner Password Credentials Grant:</h2>
<p dir="rtl">استفاده از این روش تنها در مواقعی که هیچ روش دیگری قابل اجرا نبوده و به سامانه RP  اعتماد کامل وجود دارد جایز است. در این روش  سامانه RP پس از دریافت نام کاربری و رمز عبور کاربر اقدام به ارسال موارد دریافتی  با استفاده از مکانیزم <code>POST</code>  و به صورت <code>x-www-form-urlencoded</code>  به <code>Token Endpoint</code> سامانه احراز هویت آتین کرده و در پاسخ توکن را دریافت می‌نماید.</p>
<p dir="rtl">پارامترهای لازم در این روش عبارتند از:</p>
<ol dir="rtl">
	<li><code>client_id:</code> شناسه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>client_secret:</code> گذرواژه سامانه RP تعریف شده در سامانه آتین</li>
	<li><code>grant_type:</code> این مقدار باید برابر با <code>password</code> باشد.</li>
	<li><code>username:</code> نام کاربری دریافتی از کاربر</li>
	<li><code>password:</code> گذرواژه دریافتی از کاربر</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```bash
curl --request POST \
  --url 'https://<authin_idp_address>/api/v1/oauth/token' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=YOUR_CLIENT_ID' \
  --data client_secret=YOUR_CLIENT_SECRET \
  --data grant_type=password \
  --data username=USER_USERNAME \
  --data password=USER_PASSWORD
```

**<p dir="rtl">نمونه پاسخ:</p>**

```json
{
    "id_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxIn0.rNjxxxxxxxxxxbbc",
    "access_token": "eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxCJ9.glhxxxxxxxxxxzlI",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

<h2 dir="rtl">Backchannel Logout:</h2>
<p dir="rtl">در این روش با هدف اتصال به سامانه خروج مرکزی، RP باید آدرس سرویس خروج خود را در سامانه آتین ثبت نماید. به هنگام خروج کاربر از سامانه آتین <code>Logout Token</code> به این آدرس به صورت <code>POST</code> و با فرمت <code>x-www-form-urlencoded</code> ارسال می‌شود و RP پس از اصالت سنجی توکن دریافتی اقدام به اجرای فرایند خروج کاربر و ابطال نشست مربوطه می‌نماید. لازم است RP پس از اجرای فرایند خروج کاربر یکی از پاسخ‌های HTTP زیر را به آتین برگرداند:</p>

<ol dir="rtl">
	<li><code>OK(200):</code> در صورتی که خروج با موفقیت انجام شود.</li>
	<li><code>Bad Request(400):</code>  در صورت وجود اشکال در درخواست دریافت‌شده یا اصالت سنجی توکن این پاسخ ارسال می‌شود.</li>
	<li><code>Not Implemented(501):</code> در صورتی که پس از اطمینان از صحت درخواست دریافتی، در حین اجرای عملیات خروج توسط RP، خطایی رخ دهد این پاسخ ارسال می‌شود.</li>
</ol>

**<p dir="rtl">نمونه درخواست ارسال‌شده به RP:</p>**

```bash
curl --request POST \
  --url '<backchannel_logout_uri>' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'logout_token="eyJxxxxxxxxxxiJ9.eyJxxxxxxxxxxIn0.rNjxxxxxxxxxxb1E"'
```

<h2 dir="rtl">Frontchannel Logout:</h2>
<p dir="rtl">در این روش سامانه RP  ابتدا آدرس صفحه خروج خود را در سامانه آتین ثبت می‌کند. سپس به هنگام خروج کاربر از سامانه احراز هویت مرکزی آتین، این صفحه در قالب یک <code>iframe</code> در صفحه خروج سامانه آتین بارگذاری می‌شود. لازم است به هنگام بارگذاری این صفحه، سامانه RP اقدام به ابطال نشست کاربر نماید.

<h2 dir="rtl">RP-Initiated Logout:</h2>
<p dir="rtl">در صورتی که کاربر ابتدا از سامانه RP خارج شود، لازم است سامانه RP پس از اجرای فرایند خروج، کاربر را به آدرس صفحه خروج سامانه آتین هدایت کرده تا فرایند خروج متمرکز اجرا شود. این درخواست شامل پارامترهای زیر است:</p>
<ol dir="rtl">
	<li><code>id_token_hint(اختیاری):</code> یکی از <code>ID Token</code>های دریافتی از سامانه آتین مرتبط با نشست فعلی کاربر</li>
	<li><code>post_logout_redirect_uri(اختیاری):</code> آدرس صفحه‌ای در سامانه RP که پس از اتمام فرایند خروج کاربر به آن صفحه هدایت می ‌شود. لازم است این آدرس در سامانه آتین تعریف شده باشد.</li>
	<li><code>state(اختیاری):</code> در صورت استفاده از <code>post_logout_redirect_uri</code> این مقدار عینا در پاسخ به RP بازمی‌گردد.</li>
</ol>
<blockquote dir="rtl"><strong>توجه:</strong> در صورت عدم ارائه <code>id_token_hint</code> در درخواست ارسالی، پس از اتمام فرایند خروج مرکزی در سامانه آتین، کاربر به آدرس <code>post_logout_redirect_uri</code> هدایت نخواهد شد.</blockquote>

**<p dir="rtl">نمونه درخواست:</p>**

```
https://<authin_idp_address>/logout?id_token_hint=YOUR_ID_TOKEN&post_logout_redirect_uri=YOUR_POST_LOGOUT_REDIRECT_URI&state=YOUR_STATE
```

**<p dir="rtl">نمونه پاسخ:</p>**

```
post_logout_redirect_uri?state=YOUR_STATE
```

<h2 dir="rtl">اصالت‌سنجی توکن:</h2>

**<p dir="rtl">پس از دریافت پاسخ از <code>Token Endpoint</code> لازم است ابتدا بررسی شود که نوع توکن‌های دریافتی از جنس  <code>Bearer</code> بوده و سپس مراحل ذیل جهت اصالت‌سنجی ID Token و Access Toekn صورت پذیرد.</p>**
<h3 dir="rtl">اصالت‌سنجی ساختار:</h3>
<p dir="rtl">توکن‌های ارائه شده توسط سامانه احراز هویت آتین از جنس توکن‌های <code>JWT</code> و در قابل <code>Compact Serialization</code> بوده و باید ابتدا بر اساس قواعد <a href=https://tools.ietf.org/html/rfc7519#section-7.2>RFC7519</a> اصالت‌سنجی شوند. لذا لازم است بررسی‌های زیر صورت پذیرد:</p>
<ol dir="rtl">
	<li>هر توکن <code>JWT</code> باید شامل 3 قسمت جدا شده با 2 کاراکتر <code>'.'</code> باشد.</li>
	<li>قسمت اول <code>Header</code>، قسمت دوم <code>Payload</code> و قسمت سوم <code>Signature</code> است که همگی به صورت <code>Base64Url</code> کد شده‌اند.</li>
	<li><code>Header</code> را دیکد کرده و اطمینان حاصل کنید که در فرمت <code>JSON</code> است.</li>
	<li><code>Payload</code> را دیکد کرده و اطمینان حاصل کنید که در فرمت <code>JSON</code> است.</li>
</ol>

<h3 dir="rtl">اصالت‌سنجی امضای دیجیتال:</h3>
<p dir="rtl">جهت اصالت‌سنجی <code>ID Token</code> و <code>Access Token</code> ابتدا باید لیست گواهی‌های مورد قبول سامانه احراز هویت آتین دریافت شود. سپس با استفاده از گواهی‌های ارائه شده صحت امضای دیجیتال هر یک از توکن‌ها بررسی می‌گردد و در صورتی که توکن ارائه شده با استفاده از هر یک از گواهی‌های دریافت شده اصالت‌سنجی شود امضای توکن مورد قبول است.  همچنین در صورتی که عملیات اصالت‌سنجی ناموفق باشد لازم است سامانه RP مجددا درخواست دریافت لیست گواهی‌های دیجیتال را ارسال کرده و در صورت تغییر در این لیست، لیست خود را به روزرسانی نموده و مجددا صحت امضای هر یک از توکن‌ها را بررسی نماید و در صورتی که این فرایند مجددا ناموفق باشد لازم است توکن‌ها رد شوند.</p>

**<p dir="rtl">نمونه درخواست دریافت لیست گواهی‌های دیجیتال:</p>**
```
https://<authin_idp_address>/api/v1/keys
```

**<p dir="rtl">نمونه پاسخ:</p>**
```json
{
    "keys": [
        {
            "kty": "RSA",
            "use": "SIGNATURE",
            "alg": "RS256",
            "kid": "key_id",
            "n": "seCxxxxxxxxxxCNs",
            "e": "AQAB"
        }
    ]
}
```
<p dir="rtl">مراحل لازم جهت اصالت‌سنجی امضای دیجیتال:</p>
<ol dir="rtl">
	<li>مقدار خصیصه <code>alg</code> در <code>Header</code> باید برابر با <code>RS256</code> باشد.</li>
	<li>جهت بررسی امضای دیجیتال، 2 قسمت اول توکن یعنی <code>Header</code> و <code>Payload</code> را به همان صورت <code>Base64Url</code> به وسیله <code>'.'</code>به هم چسبانده و مقدار <code>SHA256</code> آن را حساب می‌کنیم. <code dir="ltr">SHA256(Base64url-encoded Header + "." + Base64url-encoded Payload)</code></li>
	<li>حال به وسیله هر یک از گواهی‌ها بررسی میکنیم که آیا مقدار <code>Signature</code> ارائه شده در قسمت سوم توکن، امضای دیجیتال مقدار محاسبه شده در قسمت قبل است یا خیر.</li>
</ol>

<h3 dir="rtl">اصالت‌سنجی claimهای استاندارد:</h3>
<p dir="rtl">پس از اصالت‌سنجی امضای توکن لازم است محتوای claimهای استاندارد موجود در آن بررسی شود. لذا بدین صورت عمل می‌کنیم:</p>
<ol dir="rtl">
	<li>مقدار زمان انقضای توکن که در قالب <code>exp</code> و به صورت ثانیه از مبدا زمانی استاندارد ارائه شده است، باید قبل از زمان فعلی سیستم باشد.</li>
	<li>مقدار <code>iss</code> باید برابر با <code>https://authin_idp_address</code> باشد.</li>
</ol>

<h2 dir="rtl">اصالت‌سنجی ID Token:</h2>
<p dir="rtl">علاوه بر موارد مطرح شده در اصالت‌سنجی توکن لازم است موارد ذیل در رابطه با <code>ID Token</code> بررسی شود:</p>
<ol dir="rtl">
	<li>مقدار <code>aud</code> باید با <code>client_id</code> سامانه RP برابر باشد.</li>
	<li>در صورتی که روش احراز هویت <code>Implicit Flow</code> است باید مقدار <code>nonce</code> با مقدار ارائه شده در درخواست توکن یکسان بوده و سپس این مقدار از حافظه سامانه RP حذف شود.</li>
  <li>در صورتی که درخواست claimهای بیشتری را داده‌اید می‌توانید اطمینان حاصل کنید که مقادیر مورد نظر در توکن وجود دارد یا خیر.</li>
</ol>

**<p dir="rtl">نمونه ID Token:</p>**
```json
Header:
{
  "typ": "JWT",
  "alg": "RS256"
}

Payload:
{
  "token_type": "id_token",
  "iss": "https://<authin_idp_address>",
  "sub": "YOUR_USER_ID",
  "aud": "YOUR_CLIENT_ID",
  "exp": 1596475389,
  "iat": 1596471789,
  "typ": "JWT",
  "YOUR_REQUESTED_CLAIM1": "SOME_VALUE1",
  "YOUR_REQUESTED_CLAIM2": "SOME_VALUE2"
}
```

<h2 dir="rtl">اصالت‌سنجی Access Token:</h2>
<p dir="rtl">علاوه بر موارد مطرح شده در اصالت‌سنجی توکن لازم است موارد ذیل در رابطه با <code>Access Token</code> بررسی شود:</p>
<ol dir="rtl">
	<li>مقدار <code>aud</code> باید با <code>client_id</code> سامانه RP برابر باشد.</li>
  <li>در صورتی که مدیریت دسترسی سامانه RP توسط سامانه احراز هویت آتین انجام می‌شود می‌توانید لیست دسترسی‌های کاربر که با کاراکتر فاصله (space) جدا شده‌اند را در <code>scope</code> مشاهده نمایید.</li>
</ol>

**<p dir="rtl">نمونه Access Token:</p>**
```json
Header:
{
  "typ": "JWT",
  "alg": "RS256"
}

Payload:
{
  "token_type": "access_token",
  "iss": "https://<authin_idp_address>",
  "sub": "YOUR_USER_ID",
  "aud": "YOUR_CLIENT_ID",
  "exp": 1596475389,
  "iat": 1596471789,
  "scope": "profile",
  "typ": "JWT"
}
```

<h2 dir="rtl">اصالت‌سنجی Logout Token:</h2>
<p dir="rtl">پس از خروج کاربر از سامانه احراز هویت مرکزی آتین در صورتی که سامانه RP به راهکار <code>Backchannel Logout</code> متصل باشد، <code>Logout Token</code> مربوط به خروج کاربر را دریافت و فرایند خروج کاربر را اجرا می کند. در این راستا لازم است ابتدا توکن دریافتی اصالت سنجی شود:</p>
<ol dir="rtl">
  <li>ابتدا امضای دیجیتال توکن را به روش مشابهی با <code>ID Token</code> اصالت‌سنجی نمایید.</li>
  <li>مقادیر <code>aud</code> ، <code>iss</code> و <code>exp</code> را به طور مشابهی با <code>ID Token</code> صحت‌سنجی کنید.</li>
  <li>اطمینان یابید که حداقل یکی از موارد <code>sub</code> یا <code>sid</code> در توکن موجود باشد.</li>
  <li>مقدار <code>events</code> باید شامل یک مقدار <code>JSON</code> با کلید <code>http://schemas.openid.net/event/backchannel-logout</code> باشد.</li>
	<li>اطمینان یابید که توکن شامل <code>nonce</code> نباشد.</li>
</ol>

**<p dir="rtl">نمونه Logout Token:</p>**
```json
Header:
{
  "typ": "JWT",
  "alg": "RS256"
}

Payload:
{
  "token_type": "logout_token",
  "iss": "https://<authin_idp_address>",
  "sub": "YOUR_USER_ID",
  "sid": "YOUR_SESSION_ID",
  "aud": "YOUR_CLIENT_ID",
  "exp": 1596475389,
  "iat": 1596471789,
  "jti": "token_id",
  "events": {"http://schemas.openid.net/event/backchannel-logout": {}},
  "typ": "JWT"
}
```

<h2 dir="rtl">درخواست UserInfo:</h2>
<p dir="rtl">پس از دریافت <code>Access Token</code> از سامانه احراز هویت آتین می‌توان درخواست اطلاعات پروفایل کاربر را به <code>UserInfo Endpoint</code> ارائه کرد. </p>

**<p dir="rtl">نمونه درخواست:</p>**
```bash
curl -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" \
https://<authin_idp_address>/api/v1/oauth/userinfo
```

**<p dir="rtl">نمونه پاسخ:</p>**
```json
{
    "sub": "YOUR_USER_ID",
    "name": "YOUR_FULL_NAME",
    "preferred_name": "YOUR_PREFERRED_USERNAME",
    "given_name": "YOUR_GIVEN_NAME",
    "family_name": "YOUR_FAMILY_NAME"
}
```
