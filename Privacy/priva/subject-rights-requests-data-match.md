---
title: Соответствие данных запросам на права субъекта
f1.keywords:
- CSH
ms.author: chvukosw
author: chvukosw
manager: laurawi
audience: Admin
ms.topic: article
ms.service: O365-seccomp
ms.localizationpriority: medium
ms.collection:
- M365-security-compliance
- M365-priva-subject-rights-requests
search.appverid:
- MOE150
- MET150
description: Узнайте, как загрузить дополнительные сведения в Microsoft Priva о субъектах данных.
ms.openlocfilehash: 1339962a1c4dba18a1d0b21d8a2cebb17ad0f91a
ms.sourcegitcommit: f145dff5e387a8e26db2f3a2c7de125978fbacc9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2022
ms.locfileid: "62249199"
---
# <a name="data-matching-for-subject-rights-requests"></a>Соответствие данных запросам на права субъекта

С учетом совпадения данных организации могут позволить Microsoft Priva определять субъекты данных на основе точных значений предоставленных данных. Это может помочь повысить точность определения контента субъекта данных, соответствующего этим значениям данных как для внутреннего персонала, так и для внешних пользователей, с которые вы взаимодействуете. Это также упрощает необходимость вручную поставлять поля во время создания запроса на права субъекта, а также обеспечивает контекст в запросах на права субъекта и в плитке Обзор, которая демонстрирует ваши элементы с большим объемом контента субъекта данных. Дополнительные сведения об этом представлении см. в обзоре [Find and visualize personal data in Priva](priva-data-profile.md#items-with-the-most-data-subject-content).

Чтобы использовать функцию совпадения данных, необходимо быть членом группы ролей управления конфиденциальностью. Изнутри Priva в [Центр соответствия требованиям Microsoft 365](https://compliance.microsoft.com/) выберите Параметры в верхнем nav  и затем **соответствие данным**. Далее необходимо определить схему персональных данных и предоставить отправку персональных данных, как показано ниже. Обратите внимание, что можно добавлять элементы и удалять элементы, добавленные с помощью пользовательского интерфейса. Однако изменить элемент на месте из пользовательского интерфейса в настоящее время невозможно.

## <a name="prepare-for-data-import"></a>Подготовка к импорту данных

Перед определением схемы или отправки данных необходимо определить источник данных субъекта. Требуется формат файла .csv, который может быть прочитано приложением, таким как Microsoft Excel. Структурировать этот экспорт таким образом, чтобы в первом ряду появились заглавные столбцы. В этих загонах должны быть имена атрибутов схемы персональных данных. Проверьте формат данных в каждом поле. Если какой-либо из данных содержит запятые, окружай эти значения двойными кавычками, чтобы убедиться, что они не будут разделяться на отдельные поля.

## <a name="define-the-personal-data-schema"></a>Определение схемы персональных данных

Схема персональных данных описывает атрибуты субъектов данных. Upload эту схему на первой вкладке области параметров, совпадающих с данными. Необходимые файлы включают **XML-файл** схемы персональных данных и **XML-файл пакета** правил.

### <a name="personal-data-schema-xml"></a>Схема личных данных XML

Файл схемы персональных данных — это XML-файл, который определяет ожидаемые имена столбцов.

- Назови этот *файл схемыpdm.xml*.
- Определите имя каждого столбца с помощью тега Имя поля, как по примеру ниже.
- Используйте поиск = "true" для полей, которые необходимо искать, не более пяти полей. По крайней мере одно из имен полей должно быть поисковимым. Пример синтаксиса: `\<Field name="" searchable=""/>`.
- Схема персональных данных имеет раздел тегов DataStore. Четыре обязательных поля должны быть сопочены с именами полей: primaryKeyField, upnField, firstNameField, lastNameField.

В качестве примера следующий XML-файл определяет пример схемы с пятью полями, указанными в качестве поиска: PatientID, MRN, SSN, Телефон и DOB. PrimaryKeyField имеет карту PatientID, upnField — к MRN, firstNameField — к FirstName, а lastNameField — к LastName.

Вы можете скопировать, изменить и использовать наш пример.

 ```xml
<PdmSchema xmlns="http://schemas.microsoft.com/office/2020/pdm">
      <DataStore name="Patientrecords" description="Schema for patient records" version="1" primaryKeyField="PatientID" upnField="MRN" firstNameField="FirstName" lastNameField="LastName">
            <Field name="PatientID" searchable="true"/>
            <Field name="MRN" searchable="true" />
            <Field name="FirstName" />
            <Field name="LastName" />
            <Field name="SSN" searchable="true" />
            <Field name="Phone" searchable="true" />
            <Field name="DOB" searchable="true" />
            <Field name="Gender" />
            <Field name="Address" />
      </DataStore>
</PdmSchema>
 ```

### <a name="rule-package-xml"></a>XML пакета правил

При настройках пакета правил убедитесь, что вы правильно ссылались на созданный выше файл схемы персональных данных: pdm.xml. В следующем примере пакета правил XML необходимо настроить следующие поля для создания конфиденциального типа данных:

- **ID** &  RulePack **PrivacyMatch id**. Использование New-GUID для создания GUID.
- **Datastore**. В этом поле указывается хранилище данных, совпадает с данными для личных данных, которые необходимо использовать. Предопределять имя DataStore в конфигурации схемы персональных данных.
- **idMatch**. Это поле указывает на основной элемент для совпадения персональных данных.
  - **Совпадения**. Указывает поле, используемого в точном lookup. Укай имя поля для поиска из схемы персональных данных.
  - **Классификация**. В этом поле указывается совпадение конфиденциального типа, которое вызывает просмотр совпадений персональных данных. Вы можете указать имя или идентификатор GUID имеющегося встроенного или настраиваемого типа конфиденциальной информации. Чтобы избежать проблем с производительностью, если вы используете настраиваемый тип конфиденциальной информации в качестве элемента Классификация в соответствие с персональными данными, не используйте настраиваемый тип конфиденциальной информации, который будет соответствовать большому проценту контента (например, "любое число" или "любое слово с пятью буквами"). Рекомендуется добавить ключевые слова поддержки или включив форматирование в определение настраиваемого типа конфиденциальной информации классификации.
- **Match**. Это поле указывает на дополнительные доказательства, найденные в непосредственной близости от idMatch.
  - **Совпадения**. Укай любое имя поля в схеме персональных данных для DataStore.
- **Ресурс**. В этом разделе указывается имя и описание конфиденциального типа в нескольких местах.
  - **idRef**. Предоставление GUID для exactMatch ID.
  - **Описание &** имен: настраивать по мере необходимости.

В приведенном ниже примере XML пакета правил мы ссылаемся на файл pdm.xml пример с предыдущего шага, который создает схему персональных данных XML:

- **Datastore**. Имя DataStore ссылается на созданный ранее файл схемы: dataStore = "PatientRecords".
- **idMatch**. Значение idMatch ссылается на поле поиска, которое перечислены в созданном ранее файле pdm.xml: idMatch соответствует = "SSN".
  - **Классификация**. Значение классификации ссылается на существующий или настраиваемый тип конфиденциальной информации: классификация = "Номер социального обеспечения США (SSN)". (В этом случае используется существующий тип конфиденциальной информации, содержащий номер социального страхования США.)

Создайте пакет правил в формате XML (с кодированием Unicode), как в следующем коде примера. Вы можете скопировать, изменить и использовать этот пример.

 ```xml
<RulePackage xmlns="http://schemas.microsoft.com/office/2020/pdm">
  <RulePack id="fd098e03-1796-41a5-8ab6-198c93c62b21">
    <Version build="0" major="2" minor="0" revision="0" />
    <Publisher id="eb553734-8306-44b4-9ad5-c388ad970528" />
    <Details defaultLangCode="en-us">
      <LocalizedDetails langcode="en-us">
        <PublisherName>IP DLP</PublisherName>
        <Name>Health Care PDM Rulepack</Name>
        <Description>This rule package contains the Personal Data Match sensitive type for health care sensitive types.</Description>
      </LocalizedDetails>
    </Details>
  </RulePack>
  <Rules>
    <PrivacyMatch id = "E1CC861E-3FE9-4A58-82DF-4BD259EAB381" patternsProximity = "300" dataStore ="PatientRecords" recommendedConfidence = "65" >
      <Pattern confidenceLevel="65">
        <idMatch matches = "SSN" classification = "U.S. Social Security Number (SSN)" />
      </Pattern>
      <Pattern confidenceLevel="75">
        <idMatch matches = "SSN" classification = "U.S. Social Security Number (SSN)" />
        <Any minMatches ="3" maxMatches ="6">
          <match matches="PatientID" />
          <match matches="MRN"/>
          <match matches="FirstName"/>
          <match matches="LastName"/>
          <match matches="Phone"/>
          <match matches="DOB"/>
        </Any>
      </Pattern>
    </PrivacyMatch>
    <LocalizedStrings>
      <Resource idRef="E1CC861E-3FE9-4A58-82DF-4BD259EAB381">
        <Name default="true" langcode="en-us">Patient SSN Exact Match.</Name>
        <Description default="true" langcode="en-us">PDM Sensitive type for detecting Patient SSN.</Description>
      </Resource>
    </LocalizedStrings>
  </Rules>
</RulePackage>
 ```

## <a name="upload-personal-data"></a>Upload персональные данные
После определения схемы персональных данных можно выполнить отправку персональных данных на вторую вкладку страницы параметров, совпадающих с данными. При выборе **Добавить** выберите личную схему, которую вы определили на первом шаге, а затем загрузите файл, содержащий персональные данные.

Вы можете загрузить эти личные данные, выбрав локальный файл, или путем отправки URL-адреса SAS в существующее служба хранилища Microsoft Azure, содержащее файл персональных данных.
Если вы подготовили файл в качестве первого шага в этом процессе, соответствующего созданной схеме, вы можете использовать этот файл для загрузки.

## <a name="legal-disclaimer"></a>Юридический отказ

[Юридический отказ Microsoft Priva](priva-disclaimer.md)
