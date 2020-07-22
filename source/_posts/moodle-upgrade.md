---
title: moodle upgrade
date: 2020-07-22 23:43:50
categories:
tags:
---

This is an article to document the process of upgrading of moodle from 3.5 to 3.9(which is the latest version by now).

# Check plugins installed locally
``` sql
SELECT * FROM `mdl_config_plugins` where plugin like 'local%'
```

# Preparation
```
mv moodle moodle35
tar xvzf moodle-latest-3.9.tgz
```
<!--more-->

# Move files
``` bash
M35=/home/aemg/cloudclassroom/moodle35 \
M39=/home/aemg/cloudclassroom/moodle

cp $M35/config.php $M39
cp -pr $M35/availability/condition/mobileapp $M39/availability/condition/mobileapp && \
cp -pr $M35/availability/condition/paypal $M39/availability/condition/paypal

cp -pr $M35/blocks/analytics_graphs $M39/blocks/analytics_graphs && \
cp -pr $M35/blocks/completion_progress $M39/blocks/completion_progress && \
cp -pr $M35/blocks/course_modulenavigation $M39/blocks/course_modulenavigation && \
cp -pr $M35/blocks/course_search $M39/blocks/course_search && \
cp -pr $M35/blocks/coursefiles $M39/blocks/coursefiles && \
cp -pr $M35/blocks/dedication $M39/blocks/dedication && \
cp -pr $M35/blocks/heatmap $M39/blocks/heatmap && \
cp -pr $M35/blocks/quickmail $M39/blocks/quickmail

cp -pr $M35/course/format/buttons $M39/course/format/buttons && \
cp -pr $M35/course/format/grid $M39/course/format/grid && \
cp -pr $M35/course/format/onetopic $M39/course/format/onetopic && \
cp -pr $M35/course/format/topcoll $M39/course/format/topcoll

cp -pr $M35/filter/generico $M39/filter/generico && \
cp -pr $M35/grade/report/gradedist $M39/grade/report/gradedist

cp -pr $M35/lib/editor/atto/plugins/bsgrid $M39/lib/editor/atto/plugins/bsgrid && \
cp -pr $M35/lib/editor/atto/plugins/cloudvideo $M39/lib/editor/atto/plugins/cloudvideo && \
cp -pr $M35/lib/editor/atto/plugins/fontfamily $M39/lib/editor/atto/plugins/fontfamily && \
cp -pr $M35/lib/editor/atto/plugins/generico $M39/lib/editor/atto/plugins/generico && \
cp -pr $M35/lib/editor/atto/plugins/morefontcolors $M39/lib/editor/atto/plugins/morefontcolors 

cp -pr $M35/local/* $M39/local/

cp -pr $M35/login/phpuploader $M39/login/phpuploader && \
cp -pr $M35/login/test $M39/login/test && \
cp -pr $M35/login/zoom $M39/login/zoom && \
cp -pr $M35/login/Android.png $M39/login/Android.png && \
cp -pr $M35/login/Android3.4.4.png $M39/login/Android3.4.4.png && \
cp -pr $M35/login/back.php $M39/login/back.php && \
cp -pr $M35/login/change_sso_password_form.php $M39/login/change_sso_password_form.php && \
cp -pr $M35/login/demo.css $M39/login/demo.css && \
cp -pr $M35/login/forgot_sso_password_form.php $M39/login/forgot_sso_password_form.php && \
cp -pr $M35/login/form-simple-upload.php $M39/login/form-simple-upload.php && \
cp -pr $M35/login/index2.php $M39/login/index2.php && \
cp -pr $M35/login/ios.png $M39/login/ios.png && \
cp -pr $M35/login/label.png $M39/login/label.png && \
cp -pr $M35/login/label_red.png $M39/login/label_red.png && \
cp -pr $M35/login/live.php $M39/login/live.php && \
cp -pr $M35/login/live2.php $M39/login/live2.php && \
cp -pr $M35/login/logo.png $M39/login/logo.png && \
cp -pr $M35/login/phpinfo.php $M39/login/phpinfo.php && \
cp -pr $M35/login/polyv.html $M39/login/polyv.html && \
cp -pr $M35/login/server_error.txt $M39/login/server_error.txt && \
cp -pr $M35/login/sso.php $M39/login/sso.php && \
cp -pr $M35/login/sso_ss.php $M39/login/sso_ss.php && \
cp -pr $M35/login/sso_test.php $M39/login/sso_test.php && \
cp -pr $M35/login/tri.php $M39/login/tri.php

cp -pr $M35/mod/accredible $M39/mod/accredible
cp -pr $M35/mod/assign/submission/onlineaudio $M39/mod/assign/submission/onlineaudio && \
cp -pr $M35/mod/attendance $M39/mod/attendance && \
cp -pr $M35/mod/book/tool/wordimport $M39/mod/book/tool/wordimport && \
cp -pr $M35/mod/customcert $M39/mod/customcert && \
cp -pr $M35/mod/hvp $M39/mod/hvp && \
cp -pr $M35/mod/newsletter $M39/mod/newsletter && \
cp -pr $M35/mod/offlinequiz $M39/mod/offlinequiz && \
cp -pr $M35/mod/questionnaire $M39/mod/questionnaire && \
cp -pr $M35/mod/quiz/accessrule/honestycheck $M39/mod/quiz/accessrule/honestycheck && \
cp -pr $M35/mod/scheduler $M39/mod/scheduler && \
cp -pr $M35/mod/turnitintooltwo $M39/mod/turnitintooltwo && \
cp -pr $M35/mod/zoom $M39/mod/zoom 

cp -pr $M35/monitor $M39/monitor

cp -pr $M35/plagiarism/turnitin $M39/plagiarism/turnitin && \
cp -pr $M35/question/type/gapfill $M39/question/type/gapfill && \
cp -pr $M35/question/type/ordering $M39/question/type/ordering && \
cp -pr $M35/report/questionnairestats $M39/report/questionnairestats && \
cp -pr $M35/theme/lambda $M39/theme/lambda && \
cp -pr $M35/user/profile/field/multiselect $M39/user/profile/field/multiselect && \
cp -pr $M35/attendance_webservice.php $M39/attendance_webservice.php && \
cp -pr $M35/favicon.ico $M39/favicon.ico && \
cp -pr $M35/googleb7d620aa11fbf244.html $M39/googleb7d620aa11fbf244.html && \
cp -pr $M35/robots.txt $M39/robots.txt



cp -pr $M35/githash.php $M39/githash.php


rm -fr $M39/mod/customcert && \
rm -fr $M39/mod/customlabel && \
rm -fr $M39/mod/offlinequiz
```