
# Добавление кастомного поля в систему CosmoShop

Было выполнено на версии CosmoShop eMax V13.07.01


## ArtikelMaske.pm
```
/cgi-bin/cosmoshop/lib/CosmoShop/Artikel/Admin/ArtikelMaske.pm
```
В этом файле нужно добавить своё поле, я добавил вот такое

```
'is_sofort' => { class => 'IsSofort', block => 'rumpfangaben', mp => 1, sort => 11 }, 
```
- 'is_sofort' — имя поля в БД (shopartikel.is_sofort) и в форме.

- class => 'IsSofort' — класс, мы создадим его позже.

- block => 'rumpfangaben' — в какую группе отображать.

- mp => 1 — разрешает массовую обработку.

- sort => 11 — Порядок отображения.
## IsSofort.pm

```
/cgi-bin/cosmoshop/lib/CosmoShop/Artikel/Admin/ArtikelMaske
```
В данной директории добавляем свой кастомный класс, который мы написали ранее

```
package CosmoShop::Artikel::Admin::ArtikelMaske::IsSofort;
use base 'CosmoShop::Artikel::Admin::ArtikelMaske::BaseFields::Checkbox';
use strict;

# Инициализация поля
sub _init {
    my $self = shift;

    $self->{form_value} = 0;
    $self->{form_value} = 1 if $self->{Artikel} && $self->{Artikel}->isSofort;

    # Имя поля для базы и формы
    $self->{fieldname} = 'is_sofort';

    # Подпись поля в форме
    $self->{label} = 'is sofort';
    $self->{legende} = 'СОФОРТ';
}

# Генерация HTML поля
sub getField {
    my $self = shift;

    # Получаем стандартный HTML чекбокса от базового класса
    my $html = $self->SUPER::getField();

    # Добавляем ссылку для массовой обработки, если нужно
    $html .= $self->SUPER::getCopyLink if ($self->_isMassenbearbeitung && !$self->is_last() && !$self->{Artikel}->hasKombinationen());

    # Для простого булева поля дополнительных условий не нужно
    return $html;
}

1;

```
Советую взять файл похожий на наш файл по типу данных, и переписать под свой.
## Artikel.pm

```
/cgi-bin/cosmoshop/lib/CosmoShop/Artikel/Artikel.pm
```
Добавим свой геттер
```
sub isSofort {
   my $self = shift;
   return $self->{daten}->{is_sofort};
}
```

## ArtikelAdmin.pm

```
/cgi-bin/cosmoshop/lib/CosmoShop/Artikel/Admin/ArtikelAdmin.pm
```

Нужно добавить обработку нашего поля в функцию _initReqDaten()

Я это добавил вот так:

```
sub _initReqDaten {
   my $self = shift;
   my $params = shift;

   $params->{kontext} = $self->{kontext};
   $params->{parent}  = $self;

   if( !$params->{setBasisartikel}) {   # Wenn es keine BasisartikelId gibt, dann ist dieses der Hauptartikel
      $self->{daten}->{artikelid} = $self->{req}->param("artikelid");
      $self->{daten}->{artikelnr} = $self->{req}->param("artikelnr");

      # Обработка касмного поля is_sofort
      $self->{daten}->{is_sofort} = $self->{req}->param("is_sofort") ? 1 : 0;

      ...
```


## Kontext.pm

```
/cgi-bin/cosmoshop/lib/CosmoShop/Artikel/Kontext.pm
```

В данном файле нужно инициализировать наше поле и написать тип данных данного поля

```
$self->{FIELD_DEF} = {
        ...
        
        artikelid => {system=>1},
        # Кастомное поле - is_sofort
        is_sofort => { format=>'bool' },

        ...
```
