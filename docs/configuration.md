# الإعداد

<img src='../images/configuration.png'/>

عندما ستبدأ بتعلم فلاسك، ستجد أن الإعداد يبدو سهلاً للغاية. فكل ما عليك فعله هو تعريف بعض المتغيرات في الملف *config.py* وكل شيء سيعمل على ما يرام. هذه السهولة سرعان ما ستزول عندما تبدأ بإدارة الإعدادات لوضع الإنتاج. فقد تحتاج لحماية المفاتيح السرية للواجهات البرمجية أو استخدام ملفات ضبط مختلفة لبيئات مختلفة (بيئة التطوير والإنتاج على سبيل المثال). في هذا الفصل سنتحدث عن بعض ميزات فلاسك المُتقدمة التي تجعل إدارة إعدادات تطبيقك أسهل.

## الحالة البسيطة

قد لا تحتاج التطبيقات البسيطة إلى أي من هذه الميزات المعقدة. قد تحتاج فقط لوضع الملف *config.py* في جذر مستودعك واستدعاءه من ملف *app.py* أو من *yourapp/\_\_init\_\_.py* (بحسب طريقة تنظيمك للمشروع).

يجب أن يحوي الملف *config.py* على عملية إسناد قيمة لمتغير في كل سطر. عند استهلال التطبيق، تُستخدم تلك المتغيرات لضبط تطبيق فلاسك وإضافاته، ويمكن الوصول إليها باستخدام القاموس `app.config`،  مثال: `["app.config["DEBUG`.

```python
DEBUG = True # تمكين ميزة التنقيح
BCRYPT_LOG_ROUNDS = 12 # Flask-Bcrypt خيار لإضافة
MAIL_FROM_EMAIL = "robert@example.com" # للاستخدام في رسائل البريد الإلكتروني للتطبيق
```

يُمكن استخدام إعدادات الضبط (التكوين) بواسطة إطار فلاسك، أو الإضافات، أو حتى بواسطتك. في هذا المثال، يمكننا استخدام `app.config["MAIL_FROM_EMAIL"]` كلما احتجنا القيمة اﻹفتراضية لعنوان "المرسل" لاستخدامها في المعاملات البريدية، مثال: إعادة تعيين كلمة المرور. وضع معلومات كهذه في متغير إعداد يجعل من السهل تغيير قيمتها في المستقبل.

```python
# app.py أو app/__init__.py
from flask import Flask

app = Flask(__name__)
app.config.from_object('config')

# app.config["VAR_NAME"] يمكننا الآن الوصول لهذه المتغيرات باستخدام.
```

| المتغير | الشرح | نصيحة |
| ------- | ----- | ----- |
| DEBUG | يعطيك هذا الخيار بعض الأدوات المفيدة لتنقيح الأخطاء. هذا يتضمن رصة تتبع ووحدة تحكم تفاعلية مبنية على الويب. | ينبغي إسناد القيمة `True` له في وضع التطوير والقيمة `Flase` في وضع الإنتاج. |
| SECRET_KEY | هذا هو المفتاح السري المستخدم من فلاسك لتوقيع الكعكات. كما يستخدم أيضاً من إضافات مثل Flask-Bcrypt. ينبغي عليك تعريفه داخل المجلد `instance` لإبقاءه خارج نظام التحكم بالإصدارات. يمكنك قراءة المزيد حول مجلد `instance` في الفقرة التالية. | ينبغي أن يُسند لهذا الخيار قيمة عشوائية معقدة. |
| BCRYPT_LOG_ROUNDS | إذا كنت تستخدم الإضافة Flask-Bcrypt لتهشير كلمات مرور المستخدمين فعليك تحديد عدد المرات التي سيتم تنفيذ فيها الخوارزمية على كلمة المرور. إن لم تكن تستخدم هذه الإضافة فعليك البدء باستخدامها. فكلما زادت عدد الجولات، زادت معها صعوبة تخمين كلمة المرور من قبل المُهاجم. وينبغي ازدياد عدد الدورات بمرور الوقت بزيادة القدرة الحاسوبية. | سنناقش لاحقاً في الكتاب أفضل الاستخدامات لهذه الإضافة في تطبيقك. |

<blockquote>
<b>تحذير</b><br/>
تأكد من تعيين قيمة المتغير <code>DEBUG</code> إلى <code>False</code> في وضع الإنتاج. فتركه مُفعلاً يسمح للمستخدمين بتشغيل تعليمات بايثون تعسفياً على خادمك.
</blockquote>

## المجلد الحالِي

ستحتاج أحياناً لتعريف متغيرات تكوين تحتوي معلومات حساسة، وبالتالي ستحتاج لفصل هذه المتغيرات عن المتغيرات الموجودة في الملف *config.py* وإبقائها خارج المستودع. حيث ربما أنت تقوم بإخفاء بعض الأمور السريّة مثل كلمات مرور قاعدة البيانات ومفاتيح الواجهة البرمجية، أو ربما تقوم بتعريف متغيرات مخصصة لجهاز معين. لجعل الأمر سهل، يوفر فلاسك خاصية تدعى **المجلدات الحالِية**. المجلد الحالِي (instance folder) هو دليل فرعي من المستودع الجذر يحوي على ملفات إعداد مخصصة لوضع التطبيق الحالي. فنحن لا نريد إيداع هذه الملفات في نظام التحكم بالإصدارات.

```
config.py
requirements.txt
run.py
instance/
  config.py
yourapp/
  __init__.py
  models.py
  views.py
  templates/
  static/
```

### استخدام المجلد الحالِي

نستخدم التعليمة `()app.config.from_pyfile` لاستدعاء متغيرات التكوين من هذه المجلدات. إذا قمنا بتعيين `instance_relative_config=True` عند إنشاء تطبيقنا باستدعاء الصنف `()Flask` ستقوم التعليمة `()app.config.from_pyfile` بشحن الملف المُحدد من الدليل */instance*.

```python
# app.py أو app/__init__.py

app = Flask(__name__, instance_relative_config=True)
app.config.from_object('config')
app.config.from_pyfile('config.py')
```

الآن أصبح بإمكاننا تعريف المتغيرات في الملف *instance/config.py* كما كنا نفعل في الملف *config.py*. ينبغي عليك أيضاً إضافة المجلد الحالِي إلى قائمة التجاهل الخاصة بنظام التحكم بالإصدارات. للقيام بهذا باستخدام جيت، كل ما عليك هو إضافة `/instance` في الملف *gitignore.*

### المفاتيح السرية

الطبيعة الخاصة للمجلد الحالِي تجعله مكاناً مناسباً لتعريف المفاتيح التي لا تريد تتبعها بواسطة نظام التحكم بالإصدارات. هذه المفاتيح قد تتضمن مفاتيح تطبيقك السرية أو مفاتيح واجهات برمجية لطرف ثالث. هذه الخطوة مهمة جداً خاصةً إن كان تطبيقك مفتوح المصدر، أو سيصبح مفتوح المصدر في مرحلة ما في المستقبل. عادةً نطلب من المستخدمين والمساهمين الآخرين استخدام مفاتيحهم الخاصة.

```python
# instance/config.py

SECRET_KEY = 'Sm9obiBTY2hyb20ga2lja3MgYXNz'
STRIPE_API_KEY = 'SmFjb2IgS2FwbGFuLU1vc3MgaXMgYSBoZXJv'
SQLALCHEMY_DATABASE_URI= \
"postgresql://user:TWljaGHFgiBCYXJ0b3N6a2lld2ljeiEh@localhost/databasename"

```

### الإعداد بناءً على فرق صغير بين البيئات

إذا كان الفرق بين بيئة الإنتاج وبيئة التطوير بسيط فقد ترغب باستخدام المجلد الحالي للتعامل مع تغيير الإعدادات. حيث أن المتغيرات المُعرفة في *instance/config.py* يمكن أن تستبدل قيم تلك المعرفة في *config.py*. كل ما عليك القيام به هو استدعاء `()app.config.from_pyfile` بعد `()app.config.from_object`. إحدى الطرق التي يمكنك الاستفادة فيها من هذا هي تغيير طريقة إعداد تطبيقك على الأجهزة المختلفة.

```python
# config.py

DEBUG = False
SQLALCHEMY_ECHO = False


# instance/config.py
DEBUG = True
SQLALCHEMY_ECHO = True

```

بهذه الطريقة، في وضع الإنتاج، يمكننا ترك المتغيرات كما هي في *instance/config.py* فستستبدل قيمتها تلقائياً بتلك المُعرفة في *config.py*.

<blockquote>
<b>ملاحظة</b><br/>
<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>اقرأ المزيد حول <a href='http://flask-sqlalchemy.pocoo.org/latest/config/#configuration-keys'>مفاتيح إعداد</a> إضافة Flask-SQLAlchemy.</li>
</ul>
</blockquote>

## الإعداد بناءً على متغيرات البيئة

المجلد الحالِي لا ينبغي أن يكون في نظام التحكم بالإصدارات. هذا يعني أنك لن تكون قادراً على تتبع التغييرات في ملفات الضبط في هذا المجلد. قد لا يبدو الأمر مهماً لو كنت تملك متغير أو إثنين، ولكن إن كنت تملك إعدادات مضبوطة بدقة لبيئات مختلفة (بيئة الإنتاج، والتهيئة، والتطوير، إلخ.) فأنت بالتأكيد لا تريد حدوث ذلك.

يعطينا فلاسك القدرة على اختيار ملف التكوين بناءً على قيمة متغير في البيئة. هذا يعني أنه يمكننا امتلاك عدة ملفات إعداد في المستودع واختيار المناسب دائماً. طالما أننا نملك عدة ملفات إعداد، فيمكننا نقلهم إلى دليل `config` مخصص لهم.

```
requirements.txt
run.py
config/
  __init__.py # ملف فارغ، يُستخدم فقط لإخبار بايثون أن الدليل هذا حزمة
  default.py
  production.py
  development.py
  staging.py
instance/
  config.py
yourapp/
  __init__.py
  models.py
  views.py
  static/
  templates/

```

في الجدول أدناه يوجد شرح لملفات التكوين المُستخدمة.

| ملف التكوين | الشرح |
| ----------- | ----- |
| config/default.py | يحوي على القيم الافتراضية لتُستخدم على جميع البيئات أو يتم استبدالها بمتغيرات البيئة المناسبة. على سبيل المثال، قد يتم تعيين `DEBUG = False` في الملف config/default.py ويتم استبداله ب `DEBUG = True` في الملف config/development.py. |
| config/development.py | يحتوي على القيم التي تُستخدم في عملية التطوير. فقد تضع هنا عنوان قاعدة البيانات ليشير إلى خادم محلي. |
| config/production.py | يحتوي على القيم التي تستخدم في وضع الإنتاج. فقد تضع هنا عنوان قاعدة البيانات ليشير إلى خادم بعيد، على عكس العنوان المُستخدم في وضع التطوير. |
| config/staging.py | اعتماداً على عملية النشر، فقد تحتاج لخطوة تهيئة حيث تقوم باختبار التغييرات المجراة على تطبيقك على خادم يحاكي بيئة الإنتاج. على الأرجح ستحتاج لاستخدام قاعدة بيانات مختلفة هنا، كما قد تحتاج لاستبدال قيم أخرى. |

نقوم باستدعاء التعليمة `()app.config.from_envvar` لاختيار ملف الإعداد الذي نريد تحميله.

```python
# yourapp/__init__.py

app = Flask(__name__, instance_relative_config=True)

# تحميل ملف الإعداد الافتراضي
app.config.from_object('config.default')

# تحميل ملف الإعداد من المجلد الحالِي
app.config.from_pyfile('config.py')

# APP_CONFIG_FILE تحميل ملف الإعداد المحدد بواسطة متغير البيئة
# سيتم استبدال القيم المُعرفة في ملف الإعداد الافتراضي بتلك المعرفة في ملف الإعداد المُحدد بمتغير البيئة
app.config.from_envvar('APP_CONFIG_FILE')
```

يجب أن تكون القيمة المسندة لمتغير البيئة هي المسار الكامل لملف الإعداد.

تعيين قيمة متغير البيئة يعتمد على المنصة التي يعمل عليها التطبيق. فإذا كنا نعمل على خادم لينُكس يمكننا كتابة ملف نصي (shell script) لتعيين قيمة متغير البيئة وتشغيل الملف *run.py*.

```
# start.sh

export APP_CONFIG_FILE=/var/www/yourapp/config/production.py
python run.py
```

يكون الملف *start.sh* مُتغيّر تبعاً للبيئة. لذلك ينبغي عدم تتبعه بنظام التحكم بالإصدارات. على منصة هيروكو، يمكننا استخدام أداة هيروكو لتعيين قيمة متغيرات البيئة. نفس الشيء يُمكن تطبيقه على منصات PaaS (منصة كخدمة) الأخرى.

## الخلاصة

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>التطبيق البسيط قد يحتاج فقط إلى ملف إعداد واحد: <i>config.py</i>.</li>
  <li>مجلدات الحالة تساعدنا على إخفاء قيم التكوين السرية.</li>
  <li>مجلدات الحالة يُمكن أن تُستخدم لاستبدال إعدادات التطبيق في بيئة محددة.</li>
  <li>يجب استخدام متغيرات البيئة والتعليمة <code>()app.config.from_envvar</code> للتعامل مع نظام إعدادات أكثر تعقيداً قائم على نوع البيئة.</li>
</ul>
