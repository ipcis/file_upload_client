Klar, gerne! Hier ist der Code, der ein Dateipfad-Argument verwendet, um die Datei zu öffnen und hochzuladen:

```go
package main

import (
    "bytes"
    "fmt"
    "io/ioutil"
    "mime/multipart"
    "net/http"
    "os"
)

func main() {
    // Get file path from command line arguments
    if len(os.Args) < 2 {
        fmt.Println("Usage: go run main.go <file path>")
        return
    }
    filePath := os.Args[1]

    // Open file
    file, err := os.Open(filePath)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer file.Close()

    // Create HTTP request with file as body
    body := &bytes.Buffer{}
    writer := multipart.NewWriter(body)
    part, err := writer.CreateFormFile("file", filePath)
    if err != nil {
        fmt.Println(err)
        return
    }
    _, err = io.Copy(part, file)
    if err != nil {
        fmt.Println(err)
        return
    }
    writer.Close()

    req, err := http.NewRequest("POST", "https://localhost:8080/upload", body)
    if err != nil {
        fmt.Println(err)
        return
    }
    req.Header.Set("Content-Type", writer.FormDataContentType())

    // Send HTTP request
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()

    // Read HTTP response
    respBody, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Println(string(respBody))
}
```

Dieser Code liest den Dateipfad aus den Befehlszeilenargumenten aus und verwendet ihn, um die Datei zu öffnen. Anschließend wird die Datei wie zuvor in die HTTP-Anfrage kodiert und an den Server gesendet.

Beachten Sie, dass Sie den Namen des Form-Fields, das die Datei enthält, im `CreateFormFile`-Aufruf auf "file" belassen sollten, da der Server möglicherweise nur dieses Feld erwartet. Sie können jedoch den Namen der Datei, die im Formular angezeigt wird, auf den Dateinamen des Pfads festlegen, den Sie aus den Befehlszeilenargumenten erhalten haben.
