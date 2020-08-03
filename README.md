**<h1 dir="rtl">authin-doc</h1>**
<h2 dir="rtl">راهنمای اتصال از طریق پروتکل‌های OAuth 2.0 و OpenID Connect</h2>

<h2 dir="rtl">Authorization Code Flow:</h2>
<p dir="rtl">این روش احراز هویت یکی از امن‌ترین و رایج‌ترین روش‌های موجود در پروتکل‌های OpenID Connect و OAuth 2.0 است که برای استفاده توسط سامانه‌های سمت سرور طراحی شده است. فرایند احراز هویت در این روش از دو مرحله دریافت کد موقت (Authorization Code) در تعامل با کاربر و دریافت توکن توسط سرور تشکیل شده است.</p>

<h3 dir="rtl">مرحله اول:</h3>
<p dir="rtl">در این مرحل سامانه کاربر را جهت احراز هویت و دریافت Authorization Code به آدرس Authorization Endpoint به همراه پارامترهای زیر هدایت می‌کند.</p>
<ol dir="rtl">
	<li><code>client_id: </code>شناسه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>redirect_uri: </code>آدرسی که پس از احراز هویت، کاربر به همراه Authorization Code به این آدرس بازگردانده می‌شود.</li>
	<li><code>scope: </code>این پارامتر مجموعه‌ای از رشته‌های جدا شده با کاراکتر فاصله (space) است. جهت استفاده از پروتکل OpenID Connect لازم است مقدار <code>openid</code> در میان این مقادیر وجود داشته باشد. همچنین جهت دسترسی به اطلاعات پایه پروفایل کاربر از UserInfo Endpoint می‌توان مقدار <code>profile</code> را نیز به آن افزود.</li>
	<li><code>response_type: </code> مقدار این پارامتر در این روش باید برابر با <code>code</code> تنظیم شود.</li>
	<li><code>(اختیاری)state: </code>این رشته توسط سامانه احراز هویت پردازش نشده و عینا به همین شکل در پاسخ به سامانه (RP) بازگردانده می‌شود.</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```
https://<authin_idp_address>/openidauthorize?client_id=YOUR_CLIENT_ID&redirect_uri=https://YOUR_APP/callback&scope=openid profile&response_type=code&state=YOUR_STATE
```

**<p dir="rtl">نمونه پاسخ:</p>**

```
https://YOUR_APP/callback?code=YOUR_AUTHORIZATION_CODE&state=YOUR_STATE
```

<h3 dir="rtl">مرحله دوم:</h3>
<p dir="rtl">در این مرحله سامانه (RP) مقدار Authorization Code دریافتی در مرحله اول را به سمت سرور خود ارسال کرده و مستقیما از طریق سرور خود اقدام به ارسال درخواست توکن با استفاده از مکانیزم <code>POST</code> و به صورت <code>x-www-form-urlencoded</code> به Token Endpoint سامانه احراز هویت آتین می‌نماید. پارامترهای لازم در این مرحله عبارتند از:</p>
<ol dir="rtl">
	<li><code>client_id: </code>شناسه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>client_secret: </code>گذرواژه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>grant_type: </code>این مقدار باید برابر با <code>authorization_code</code> باشد.</li>
	<li><code>code: </code>مقدار Authorization Code دریافتی در مرحله اول</li>
	<li><code>redirect_uri: </code>این مقدار باید عینا با مقدار ارائه شده در مرحله اول یکسان باشد.</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```
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
<p dir="rtl">پس  از اتمام فرایند احراز هویت یکی از توکن‌های دریافتی Refresh Token است که سامانه RP با ارائه این توکن به سامانه احراز هویت آتین می‌تواند بدون مداخله کاربر توکن جدیدی دریافت کند. مکانیزم درخواست Refresh Token بدین صورت است که در صورتی که عمر توکن فعلی کاربر به اتمام رسیده و توکن منقضی شده باشد،  RP با ارائه Refresh Token دریافتی، توکن جدیدی را به دست می‌آورد. لازم به ذکر است که به هنگام اجرای قرایند خروج از سامانه RP، باید Refresh Token  پاک شود.</p>
<p dir="rtl">سامانه RP جهت دریافت توکن جدید باید درخواست Refresh Token را با استفاده از مکانیزم <code>POST</code> و به صورت <code>x-www-form-urlencoded</code> به Token Endpoint سامانه احراز هویت آتین ارائه دهد. پارامترهای این درخواست عبارتند  از:</p>
<ol dir="rtl">
	<li><code>client_id: </code>شناسه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>client_secret: </code>گذرواژه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>grant_type: </code>این مقدار باید برابر با <code>refresh_token</code> باشد.</li>
	<li><code>refresh_token: </code>مقدار Refresh Token دریافتی در هنگام احراز هویت کاربر</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```
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
<p dir="rtl">استفاده از این روش تنها در مواقعی که هیچ روش دیگری قابل اجرا نبوده و به سامانه RP اعتماد کامل وجود دارد جایز است. در این روش RP پس از دریافت نام کاربری و رمز عبور کاربر اقدام به ارسال موارد دریافتی  با استفاده از مکانیزم <code>POST</code>  و به صورت <code>x-www-form-urlencoded</code>  به Token Endpoint سامانه احراز هویت آتین کرده و در پاسخ توکن را دریافت می‌نماید.</p>
<p dir="rtl">پارامترهای لازم در این روش عبارتند از:</p>
<ol dir="rtl">
	<li><code>client_id: </code>شناسه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>client_secret: </code>گذرواژه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>grant_type: </code>این مقدار باید برابر با <code>password</code> باشد.</li>
	<li><code>username: </code>نام کاربری دریافتی از کاربر</li>
	<li><code>password: </code>گذرواژه دریافتی از کاربر</li>
</ol>

**<p dir="rtl">نمونه درخواست:</p>**

```
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

<h2 dir="rtl">اصالت‌سنجی توکن:</h2>

**<p dir="rtl">پس از دریافت پاسخ از <code>Token Endpoint</code> لازم است ابتدا بررسی شود که نوع توکن‌های دریافتی از جنس  <code>Bearer</code> بوده و سپس مراحل ذیل جهت اصالت‌سنجی ID Token و Access Toekn صورت پذیرد.</p>**
<h3 dir="rtl">اصالت‌سنجی ساختار:</h3>
<p dir="rtl">توکن‌های ارائه شده توسط سامانه احراز هویت آتین از جنس توکن‌های <code>JWT</code> و در قابل <code>Compact Serialization</code> بوده و باید ابتدا بر اساس قواعد <a href=https://tools.ietf.org/html/rfc7519#section-7.2>RFC7519</a> اصالت‌سنجی شوند. لذا لازم است بررسی‌های زیر صورت پذیرد:</p>
<ol dir="rtl">
	<li>هر توکن JWT باید شامل 3 قسمت جدا شده با 2 کاراکتر <code>'.'</code> باشد.</li>
	<li>قسمت اول Header، قسمت دوم Payload و قسمت سوم Signature است که همگی به صورت Base64Url کد شده‌اند.</li>
	<li>Header را دیکد کرده و اطمینان حاصل کنید که در فرمت JSON است.</li>
	<li>Payload را دیکد کرده و اطمینان حاصل کنید که در فرمت JSON است.</li>
</ol>

<h3 dir="rtl">اصالت‌سنجی امضای دیجیتال:</h3>
<p dir="rtl">جهت اصالت‌سنجی ID Token و Access Token ابتدا باید لیست گواهی‌های مورد قبول سامانه احراز هویت آتین دریافت شود. سپس با استفاده از گواهی‌های ارائه شده صحت امضای دیجیتال هر یک از توکن‌ها بررسی می‌گردد و در صورتی که توکن ارائه شده با استفاده از هر یک از گواهی‌های دریافت شده اصالت‌سنجی شود امضای توکن مورد قبول است.  همچنین در صورتی که عملیات اصالت‌سنجی ناموفق باشد لازم است سامانه RP مجددا درخواست دریافت لیست گواهی‌های دیجیتال را ارسال کرده و در صورت تغییر در این لیست، لیست خود را به روزرسانی نموده و مجددا صحت امضای هر یک از توکن‌ها را بررسی نماید و در صورتی که این فرایند مجددا ناموفق باشد لازم است توکن‌ها رد شوند.</p>

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
	<li>جهت بررسی امضای دیجیتال، 2 قسمت اول توکن یعنی <code>Header</code> و <code>Payload</code> را به همان صورت Base64Url به وسیله <code>'.'</code>به هم چسبانده و مقدار SHA256 آن را حساب می‌کنیم. <code dir="ltr">SHA256(Base64url-encoded Header + "." + Base64url-encoded Payload)</code></li>
	<li>حال به وسیله هر یک از گواهی‌ها بررسی میکنیم که آیا مقدار <code>Signature</code> ارائه شده در قسمت سوم توکن، امضای دیجیتال مقدار محاسبه شده در قسمت قبل هست یا خیر.</li>
</ol>

<h3 dir="rtl">اصالت‌سنجی claimهای استاندارد:</h3>
<p dir="rtl">پس از اصالت‌سنجی امضای توکن لازم است محتوای claimهای استاندارد موجود در آن بررسی شود. لذا بدین صورت عمل می‌کنیم:</p>
<ol dir="rtl">
	<li>مقدار زمان انقضای توکن که در قالب <code>exp</code> و به صورت ثانیه از مبدا زمانی استاندارد ارائه شده است، باید قبل از زمان فعلی سیستم باشد.</li>
	<li>مقدار <code>iss</code> باید برابر با <code>https://authin_idp_address</code> باشد.</li>
</ol>

<h2 dir="rtl">اصالت‌سنجی ID Token:</h2>
<p dir="rtl">علاوه بر موارد مطرح شده در اصالت‌سنجی توکن لازم است موارد ذیل در رابطه با ID Token بررسی شود:</p>
<ol dir="rtl">
	<li>مقدار <code>aud</code> باید به <code>client_id</code> سامانه RP برابر باشد.</li>
	<li>در صورتی که روش احراز هویت <code>Implicit Flow</code> است باید مقدار <code>nonce</code> با مقدار ارائه شده در درخواست توکن یکسان بوده و سپس این مقدار از حافظه سامانه RP حذف شود.</li>
  <li>در صورتی که درخواست claimهای بیشتری را داده‌اید می‌توانید اطمینان حاصل کنید که مقادیر مورد نظر در توکن وجود دارد یا خیر.</li>
</ol>

<h2 dir="rtl">اصالت‌سنجی Access Token:</h2>
<p dir="rtl">علاوه بر موارد مطرح شده در اصالت‌سنجی توکن لازم است موارد ذیل در رابطه با Access Token بررسی شود:</p>
<ol dir="rtl">
	<li>مقدار <code>aud</code> باید به <code>client_id</code> سامانه RP برابر باشد.</li>
  <li>در صورتی که مدیریت دسترسی سامانه RP توسط سامانه احراز هویت آتین انجام می‌شود می‌توانید لیست دسترسی‌های کاربر که با کاراکتر فاصله جدا شده‌اند را در <code>scope</code> مشاهده نمایید.</li>
</ol>