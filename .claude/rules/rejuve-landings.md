# Добавление страницы в REjuve Portal

## Что нужно сделать (3 шага)

### Шаг 1 — скопировать HTML-файл
```
cp <source>.html C:/dev/h2t-landings/rejuve/<filename>.html
```
Имя файла: kebab-case, без даты-префикса, max 40 символов.

### Шаг 2 — добавить запись в реестр
Открыть `C:/dev/h2t-landings/rejuve/index.html`.  
Найти блок `// ── ДОБАВИТЬ НОВУЮ СТРАНИЦУ` в конце массива `PAGES`.  
Раскомментировать шаблон и заполнить поля:

```js
,{
  cat: 'research',          // proposal | analytics | research | appendix
  title: 'Название',
  desc: 'Одна-две строки: что внутри.',
  file: 'filename.html',    // имя файла из шага 1
  date: 'YYYY-MM-DD',       // дата создания документа
  audience: 'internal'      // client (показывается клиенту) | internal
}
```

### Шаг 3 — коммит и пуш
```
git -C C:/dev/h2t-landings add rejuve/<filename>.html rejuve/index.html
git -C C:/dev/h2t-landings commit -m "rejuve: add <title>"
git -C C:/dev/h2t-landings push origin main
```

## Категории

| cat | Когда использовать |
|---|---|
| `proposal` | Презентации и офферы для клиента |
| `analytics` | Данные SimplyBook, отчёты по KPI |
| `research` | Research по рынку, партнёрам, конкурентам |
| `appendix` | Технические приложения, декомпозиция |

## Правила

- `audience: 'client'` — только если документ предназначен для показа Kristina / Petr.  
- `audience: 'internal'` — всё остальное (research, analytics, рабочие материалы).  
- Файл в `rejuve/` должен быть самодостаточным HTML (без зависимостей вне папки).  
- Не трогать рендер-код ниже строки `// Ниже — рендер. Менять не нужно.`

## Портал live

https://lichtpfad.github.io/h2t-landings/rejuve/
