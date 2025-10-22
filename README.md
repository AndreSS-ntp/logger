# logger

Простой, потокобезопасный логгер на Go без внешних зависимостей.
Пишет в `io.Writer` (по умолчанию — `os.Stdout`), поддерживает уровни `INFO`, `WARN`, `ERROR`, интегрируется с `context.Context` и HTTP-хэндлерами.

## Установка

```bash
go get github.com/AndreSS-ntp/logger
```

## Быстрый старт

```go
l := logger.NewLogger()
l.Info(context.Background(), "app started")
l.Warn(context.Background(), "cache miss")
l.Error(context.Background(), "cannot connect to db")
```

Вывод (UTC, RFC3339):

```
2025-10-22T12:34:56Z [INFO] app started
2025-10-22T12:34:56Z [WARN] cache miss
2025-10-22T12:34:56Z [ERROR] cannot connect to db
```

## Уровни логирования

```go
l.SetLevel(logger.LevelWarn) // теперь INFO подавляется
l.Info(ctx, "won't be printed")
l.Warn(ctx, "visible")
l.Error(ctx, "visible")
```

Доступные уровни:

* `LevelInfo`
* `LevelWarn`
* `LevelError`

## Перенаправление вывода

```go
f, _ := os.Create("app.log")
defer f.Close()

l.SetWriter(f)     // писать в файл
l.SetWriter(nil)   // вернуть вывод на Stdout
```

Любой `io.Writer` подходит: файл, буфер, сеть и т. п.

## Контекст и логгер по умолчанию

Модуль несёт «логгер по умолчанию» и умеет класть/доставать логгер из контекста.

```go
// Привязать логгер к контексту
ctx := logger.WithLogger(context.Background(), l)

// Достать логгер (если нет в контексте — вернётся defaultLogger)
log := logger.FromContext(ctx)
log.Info(ctx, "hello from context")
```

Это удобно для библиотек/хэндлеров, которые не знают о конкретной реализации логгера.

## Интеграция с HTTP

Обёртка прокидывает логгер в контекст запроса:

```go
mux := http.NewServeMux()

mux.Handle("/ping", logger.HandlerWithLogger(l, func(w http.ResponseWriter, r *http.Request) {
    log := logger.FromContext(r.Context())
    log.Info(r.Context(), "ping requested")
    w.WriteHeader(http.StatusOK)
    _, _ = w.Write([]byte("pong"))
}))

_ = http.ListenAndServe(":8080", mux)
```

## Формат сообщения

```
<TIMESTAMP UTC RFC3339> [<LEVEL>] <message>\n
```

Пример: `2025-10-22T12:34:56Z [WARN] cache miss`

## Потокобезопасность

Все операции на логгере защищены мьютексом: можно безопасно менять уровень/вывод и писать логи из разных горутин.

## Интерфейс

```go
type Logger interface {
    Info(ctx context.Context, msg string)
    Warn(ctx context.Context, msg string)
    Error(ctx context.Context, msg string)
    SetWriter(w io.Writer)
    SetLevel(level Level)
}
```

## Заметки

* Без зависимостей, только стандартная библиотека.
* Timestamp всегда в UTC.
* Если уровень ниже установленного — запись пропускается.

## Лицензия

MIT.