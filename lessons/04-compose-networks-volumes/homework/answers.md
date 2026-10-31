

1.** `stats` מגיע ל־backend דרך רשת Docker המשותפת, באמצעות שם השירות `backend:5000`. הדפדפן מגיע ל־backend דרך Nginx, שמנתב את הבקשות אליו.

**2.** `docker compose down` מסיר את הקונטיינרים אך שומר על הנתונים ב־volumes. `docker compose down -v` מסיר גם volumes  ולכן הנתונים שבהם נמחקים; 

**3.** זהו `bind mount`, כי מצוין נתיב מקומי (`../sql/init.sql`) שמחובר לנתיב בתוך הקונטיינר. שינוי הקובץ לאחר יצירת המסד לא יריץ מחדש את סקריפט האתחול, כי הוא מופעל רק ביצירת מסד חדש.

**4.** שם הרשת ושם ה־volume נוצרים אוטומטית על ידי Compose,7846b9c40d8b   docker_default   bridge    local /// 
DRIVER    VOLUME NAME
local     2b358c32f816d88d7da4b5c11367aceb645c80760268a80850bd4b9b2ac7ff74
local     d9e5808e1612ed90feed30303f5670dff97095fb5fe89473c2048de92459b72b
local     docker_postgres-data

**5.** ב־image הסופי יש Nginx, קובצי האתר שנבנו וקובץ ההגדרות, ואין בו Node.js או `build.js`. אין צורך ב־`CMD` כי תמונת Nginx כבר כוללת פקודת הפעלה. ל־stats לא צריך Multi-stage כי אין לו שלב בנייה נפרד.

**6.** `kitchen` הוא 244MB, `multi` הוא 92.7MB ו־`single` הוא 92.8MB. `multi` כמעט בגודל של `single` כי שניהם מבוססים על Nginx ומכילים כמעט את אותם קבצים.

**7.** הבקשה `/api/stats` מתאימה ל־`location /api/`, אבל אם יש לה `location` ספציפי יותר, הוא יקבל עדיפות. בגלל הלוכסן בסוף `http://backend:5000/`, Nginx מחליף את הקידומת `/api/` ב־`/`, ולכן `/api/products` מגיע ל־backend בתור `/products`.