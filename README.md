**<h1 dir="rtl">authin-doc</h1>**
<h2 dir="rtl">راهنمای اتصال از طریق پروتکل‌های OAuth 2.0 و OpenID Connect</h2>

<h2 dir="rtl">Authorization Code Flow:</h2>
<p dir="rtl">این روش احراز هویت یکی از امن‌ترین و رایج‌ترین روش‌های موجود در پروتکل‌های OpenID Connect و OAuth 2.0 است که برای استفاده توسط سامانه‌های سمت سرور طراحی شده است. فرایند احراز هویت در این روش از دو مرحله دریافت کد موقت (Authorization Code) در تعامل با کاربر و دریافت توکن توسط سرور تشکیل شده است.</p>

<h3 dir="rtl">مرحله اول:</h3>
<p dir="rtl">در این مرحل سامانه کاربر را جهت احراز هویت و دریافت Authorization Code به آدرس Authorization Endpoint به همراه پارامترهای زیر هدایت می‌کند.</p>
<ol dir="rtl">
	<li><code>client_id: </code>شناسه سامانه (RP) تعریف شده در سامانه آتین</li>
	<li><code>redirect_uri: </code>آدرسی که پس از احراز هویت، کاربر به همراه Authorization Code به این آدرس بازگردانده می‌شود.</li>
	<li><code>scope: </code>این پاراکتر مجموعه‌ای از رشته‌های جدا شده با کاراکتر فاصله (space) است. جهت استفاده از پروتکل OpenID Connect لازم است مقدار <code>openid</code> در میان این مقادیر وجود داشته باشد. همچنین جهت دسترسی به اطلاعات پایه پروفایل کاربر از UserInfo Endpoint می‌توان مقدار <code>profile</code> را نیز به آن افزود.</li>
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
<h1>توضیحات اضافه شود!!!!!!!!!!!!!!!</h1>
<h1>claim چی؟</h1>

<h3 dir="rtl">به روز رسانی توکن (Refresh Token):</h3>
<p dir="rtl">پس  از اتمام فرایند احراز هویت یکی از توکن‌های دریافتی Refresh Token است که سامانه RP با ارائه این توکن به آتین می‌تواند بدون مداخله کاربر توکن جدیدی دریافت کند. مکانیزم درخواست Refresh Token بدین صورت است که در صورتی که عمر توکن فعلی کاربر به اتمام رسیده و توکن منقضی شده باشد،  RP با ارائه Refresh Token دریافتی، توکن جدیدی را به دست می‌آورد. لازم به ذکر است که به هنگام اجرای قرایند خروج از سامانه RP، باید Refresh Token  پاک شود.</p>
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
<h1>توضیحات اضافه شود!!!!!!!!!!!!!!!</h1>

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
<h1>توضیحات اضافه شود!!!!!!!!!!!!!!!</h1>

<h2 dir="rtl">اصالت سنجی توکن:</h2>
<p dir="rtl">پس از دریافت توکن‌ها لازم است ابتدا فرایند اصالت سنجی (Token Validation) انجام شود.  اولین گام در این راستا بررسی امضای دیجیتال توکن است. به همین منظور ابتدا باید لیست گواهی‌های مورد قبول سامانه احراز هویت آتین دریافت شود. سپس با استفاده از گواهی‌های ارائه شده صحت امضای دیجیتال هر یک از توکن‌ها بررسی می‌گردد و در صورتی که توکن ارائه شده با استفاده از هر یک از گواهی‌های دریافت شده اصالت‌سنجی شود امضای توکن مورد قبول است.  همچنین در صورتی که عملیات اصالت‌سنجی ناموفق باشد لازم است سامانه RP مجددا درخواست دریافت لیست گواهی‌های دیجیتال را ارسال کرده و در صورت تغییر در این لیست، لیست خود را به روزرسانی نموده و مجددا صحت امضای هر یک از توکن‌ها را بررسی نماید و در صورتی که این فرایند مجددا ناموفق باشد لازم است توکن‌ها رد شوند.</p>

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
<h3 dir="rtl">اصالت سنجی Access Token:</h3>


<h3 dir="rtl">اصالت سنجی ID Token:</h3>























<h2 dir="rtl">راهنمای استفاده از Authin.Api.Sdk در NET.</h2>

<p dir="rtl">به منظور احراز هویت و احراز دسترسی کاربر، می‌بایست روال زیر به ترتیب انجام شود:</p>

**<p dir="rtl">1. هدایت کاربر به آدرس سامانه احراز هویت به روش زیر:</p>**


```csharp
var authorizationRequest = AuthorizationCodeRequest.GetBuilder()
    .SetBaseUrl("IAM_BASE_ADDRESS")                         	(1)
    .SetClientId("YOUR_CLIENT_ID")                          	(2)
    .SetRedirectUri("YOUR_REDIRECT_URI")                    	(3)
    .SetResponseType("code")                                	(4)
    .AddScopes(new List<string>() {"openid", "profile"})    	(5)
    .AddIdTokenClaim("username")                            	(6)
    .AddIdTokenClaim("lastname")                            	(7)
    .SetState("some_data")                                  	(8)
    .Build();

var authorizationResult = await authorizationRequest.Execute();	(9)
```
<ol dir="rtl">
	<li>آدرس سامانه احراز هویت مثال:  <code>https://demo.authin.ir</code></li>
	<li><code>client_id</code> سامانه شما در سامانه احراز هویت (این پارامتر را از ما دریافت می‌کنید)</li>
	<li><code>redirect_uri</code>ای که در تنظیمات سامانه شما در سامانه احراز هویت ثبت شده است. بعد از اتمام فرآیند احراز هویت، کاربر به همراه یک کد که اصطلاحا <code>authorization code</code> نام دارد به آدرس مذکور هدایت می‌شود. برای اطلاعات بیشتر به <a href="https://www.oauth.com/oauth2-servers/redirect-uris/">Redirect URIs</a> رجوع کنید. به طور مثال اگر <code>redirect_uri</code> شما برابر با <code>htt://my.domain.com/Account/ExternalLoginCallback</code> باشد، کاربر به آدرس <code>htt://my.domain.com/Account/ExternalLoginCallback?code=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</code> هدایت می‌شود. پارامتر <code>code</code> برای مرحله بعد لازم است.</li>
	<li>طبق پروتکل، نوع جوابی که خواهان دریافت آن هستیم را معین می‌کند که در این مرحله <code>"code"</code> می‌باشد.</li>
	<li>لیست دسترسی‌هایی که می‌خواهید بر روی توکن کاربر نوشته شود را تعیین می‌کنید.
		<ul>
			<li>خصیصه‌ی <code>openid</code> همواره باید درخواست داده شود.</li>
			<li>اگر غیر از مورد قبل، هیچ دسترسی دیگری را درخواست ندهید، تمامی دسترسی‌های موجود کاربر بر روی توکن نوشته می‌شود.</li>
			<li>اگر هر دسترسی خاصی را که مد نظر دارید درخواست دهید، فقط همان دسترسی‌ها بر روی توکن
    کاربر نوشته می‌شوند.</li>
		</ul>
	</li>
	<li>خصیصه‌های پروفایل کاربر را که می‌خواهید به درخواست اضافه کنید. این خصیصه بر روی <code>id_token</code> کاربر نوشته می‌شود.</li>
	<li>همانند مورد قبل، هر خصیصه‌ای را که مد نظر دارید، می‌توانید به درخواست اضافه کنید.</li>
    <li>پارامتر <code>state</code> این امکان را به شما می‌دهد تا وضعیت (state) قبلی سیستم خود را بازیابی کنید. این پارامتر بعد از بازهدایت کاربر به <code>redirect_uri</code> عینا همراه url ارسال می‌شود. برای مطالعه بیشتر به <a href="https://auth0.com/docs/protocols/oauth2/oauth-state">State Parameter</a> رجوع کنید.</li>
	<li>نتیجه، آدرسی است که به منظور احراز هویت کاربر، باید او را به آدرس مذکور هدایت کنید.</li>
</ol>
   
<br/>	 

**<p dir="rtl">2. در مرحله دوم، بعد از این که کاربر احراز هویت شد و به سامانه شما هدایت شد، نیاز است تا از سامانه احراز هویت، درخواست token برای کاربر مذکور بدهید. بدین منظور به روش زیر عمل کنید:</p>**

```csharp
var tokenRequest = TokenRequest.GetBuilder()
		.SetBaseUrl("IAM_BASE_ADDRESS")
		.SetClientId("YOUR_CLIENT_ID")          (1)
		.SetClientSecret("YOUR_CLIENT_SECRET")  (2)
		.SetRedirectUri("YOUR_REDIRECT_URI")    (3)
		.SetGrantType("authorization_code")     (4)
		.SetCode(code)                          (5)
		.Build();

var tokenResult = await tokenRequest.Execute();	(6)
```
<ol dir="rtl">
	<li><code>client_id</code> سامانه شما در سامانه احراز هویت (این پارامتر را از ما دریافت می‌کنید)</li>
	<li><code>client_secret</code> سامانه شما در سامانه احراز هویت (این پارامتر را از ما دریافت می‌کنید)</li>
	<li>مقدار <code>redirect_uri</code> که در مرحله قبل قرار دادید. توجه کنید باید این مقدار در هر دو مرحله یکی باشند.</li>
	<li>چارچوب <code>grant type</code>، <code>OAuth</code>های مختلفی را برای کاربردهای مختلف مشخص می‌کند. یکی از آن‌ها <code>authorization_code</code> است که برای درخواست توکن در ازای ارائه <code>Authorization Code</code> مورد استفاده قرار می‌گیرد. این مقدار را در این مرحله <code>"authorization_code"</code> قرار دهید. برای اطلاعات بیشتر به <a href="https://www.oauth.com/oauth2-servers/server-side-apps/authorization-code/">Authorization Code Grant</a> رجوع کنید.</li>
	<li><code>code</code>ای را که در زیر بخش ۳ از مرحله قبل دریافت کردید در این قسمت قرار دهید.</li>
	<li>پاسخ شامل پارامترهای <code>access_token</code>، <code>id_token</code> و <code>refresh_token</code> است.</li>
</ol>

<br/>	 

**<p dir="rtl">3. بعد از دریافت توکن لازم است تا اقدام به صحت سنجی توکن‌های دریافتی کنید. این کار دارای ۲ مرحله است:</p>**
<p dir="rtl"> 1. مرحله اول: دریافت کلید عمومی</p>

```csharp
var jwksRequest = JwksRequest.GetBuilder()
		.SetBaseUrl("IAM_BASE_ADDRESS")
		.Build();

var jwksResult = await jwksRequest.Execute();
```

<p dir="rtl"> 2. مرحله دوم: صحت سنجی و تجزیه توکن</p>

```csharp
var decodedJwt = TokenValidator.Validate(
			tokenResult.AccessToken,    (1)
			jwksResult,                 (2)
			"ISSUER",                   (3)
			"AUDIENCE"                  (4)
);
```
<ol dir="rtl">
	<li><code>access_token</code> دریافتی در مرحله ۲</li>
	<li>کلید عمومی دریافتی در مرحله ۱ از مراحل صحت سنجی</li>
	<li>صادر کننده توکن مثال <code>https://www.authin.ir</code></li>
	<li>شناسه گیرنده توکن (کسی که توکن برای او صادر شده)</li>
</ol>
	
<ul dir="rtl">
	<li>جواب دریافتی شامل فقره‌های اطلاعاتی موجود در هر توکن می‌باشد. به طور مثال <code>scope</code>هایی که در مرحله ۱ درخواست داده شده‌اند در <code>access_token</code> هستند و یا <code>claim</code>هایی  که در مرحله ۱ در زیر بخش‌های شماره ۶ و ۷ اضافه شده‌اند را در تجزیه <code>id_token</code> دریافت خواهید کرد.
	</li>
</ul>

<blockquote dir="rtl"> توجه: به هیچ عنوان بدون صحت‌سنجی، توکن‌های دریافتی را استفاده نکنید. توکن‌ها به معنی اعتبارنامه دسترسی به سامانه شما هستند. </blockquote>

**<p dir="rtl">4. برای دریافت اطلاعات کاربر که درخواست آن را در مرحله ۱ در <code>scope</code>ها داده‌اید، به روش زیر عمل کنید:</p>**

```csharp
var userInfoRequest = UserInfoRequest.GetBuilder()
		.SetBaseUrl("IAM_BASE_ADDRESS")
		.SetMethod(Method.Get)                      (1)
		.SetAccessToken(tokenResult.AccessToken)    (2)
		.Build();

var userInfoResult = await userInfoRequest.Execute();
```
<ol dir="rtl">
	<li>نوع درخواست که می‌تواند <code>GET</code> یا <code>POST</code> باشد.</li>
	<li><code>access_token</code> دریافتی در مرحله ۲.</li>
</ol>
