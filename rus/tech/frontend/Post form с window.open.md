четверг, 19 июня 2008 г. в 17:13:56

Я сам не люблю popup-окна, но иногда клиенту очень уж хочется. Публиковать же форму в такое окошко имеет смысл например при работе с отчётами - в основном окошке выделяются данные, а в открывающемся показывается список готовый для распечатки c window.print()

Решение простое:

`<form method='post' action='' target="foo"   onSubmit="window.open('', 'foo', 'width=850,height=400,status=yes,resizable=yes,scrollbars=yes')">`

Если же подобное открытие формы должно зависеть от других элементов, например checkbox'а, то на jQuery это выглядит так:

`$('#report_form').submit(function(){           if ($('#csv_radio').attr('checked')!=true){               this.target="foo";               window.open('', 'foo', 'width=950,height=400,status=yes,resizable=yes,scrollbars=yes');           }           else this.target="_self";       });`