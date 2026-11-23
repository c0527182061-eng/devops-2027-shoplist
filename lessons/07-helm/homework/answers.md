



## 1. מה ההבדל בין Chart ל־Release?

Chart הוא תבנית של האפליקציה. Release הוא התקנה מסוימת של ה־Chart בקלאסטר. בשיעור, `./helm/shoplist` הוא ה־Chart, ו־`shop` הוא שם ה־Release שהתקנו ב־namespace בשם `shop`.

## 2. למה הסרנו את `namespace.yml` מה־Chart, ומה מחליף אותו?

הסרנו אותו כדי שאותו Chart יוכל לשמש להתקנה בכל namespace שנבחר. בפקודת ההתקנה `-n shop`  `{{ .Release.Namespace }}` מכניס למשאבים את שם ה־namespace שנבחר.

## 3. מאיפה הגיעו הערכים שלא כתבנו ב־`values-big.yaml`?

Helm מיזג את `values-big.yaml` עם ברירות המחדל שב־`values.yaml`. הקובץ `values-big.yaml` שינה רק את מספר העותקים של `backend` ושל `stats`. שאר הערכים נשארו מ־`values.yaml`, למשל `frontend.nodePort: 30080`.

## 4. למה האפליקציה עבדה גם כשה־image היה שבור?
-האפלקציה עבדה לאחר הפקודה של שינוי התג לBACKEND כי הוא מוריד POD רק לאחר שאחרים עלו בהצלחה-וכאן הוא נכשל


## 5. למה אחרי ה־rollback יש ארבעה Revisions?

Helm מתעד את ה־rollback כפעולה חדשה בהיסטוריה. Revision 1 הוא ההתקנה, Revision 2 הוא העדכון עם `values-big.yaml`, ו־Revision 3 הוא העדכון עם התגית השגויה. ה־rollback מחיל מחדש את ההגדרות של Revision 2 ושומר את הפעולה כ־Revision 4, עם התיאור `Rollback to 2`.

## 6. מה היה קורה לשלושת המוצרים בלי `helm.sh/resource-policy: keep`?

בזמן `helm uninstall`, גם ה־PVC של PostgreSQL היה נמחק. בהתקנה מחדש ה־Chart היה יוצר PVC חדש, ובסביבת התרגיל PostgreSQL היה מתחיל עם אחסון חדש וריק. לכן שלושת המוצרים לא היו מופיעים אחרי ההתקנה מחדש.

## 7. מה קרה כשכתבנו `{{ .Values.backend.replica }}`?

ב־`values.yaml` המפתח נקרא `replicas` ברבים. בגלל הטעות Helm היה יכול להפיק שורה `replicas:` בלי ערך. Kubernetes היה משתמש בברירת המחדל של עותק אחד, כך שגם עם `values-big.yaml` היינו רואות Pod אחד של ה־backend במקום שלושה. אפשר לגלות את זה עם `helm template shop ./helm/shoplist -n shop -f ./helm/shoplist/values-big.yaml` ולחפש את השורה הריקה, או לבדוק את מספר ה־Pods עם `kubectl get pods -n shop`.

## 8. מה קורה כש־PostgreSQL חורג ממגבלת הזיכרון או ה־CPU?

אם ה־container מנסה לצרוך יותר מ־`512Mi` זיכרון, הקרנל יכול להרוג את התהליך. במקרה כזה אפשר לראות `OOMKilled`, ו־Kubernetes ינסה להפעיל מחדש את ה־container. אם הוא מנסה להשתמש ביותר מליבת CPU אחת, הקרנל מגביל את זמן המעבד שהוא מקבל באמצעות CPU throttling, ולכן הוא עלול לעבוד לאט יותר. זמן מעבד אפשר לדחות ולהקצות בהמשך, ואילו חריגה ממגבלת הזיכרון יכולה לחייב שחרור זיכרון באמצעות סיום תהליך.

## 9. למה הגיבוי הוא CronJob?

CronJob יוצר Job בכל מועד שנקבע בלוח הזמנים. ה־Job מריץ את הגיבוי פעם אחת ומסתיים. Deployment מיועד להחזיק Pods פעילים לאורך זמן. אם היינו מריצות בו את אותה פקודת גיבוי חד־פעמית, ה־container היה מופעל שוב אחרי סיום הפקודה, והגיבוי היה מנסה לרוץ שוב ושוב במקום פעם בלילה.