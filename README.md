<h1> Web Note 📒 </h1>

## Запуск програмы 🏃
* Вы можете запустить программу введя следующую команду: `go run note.go`


---
## Реализация программы ⚡
_Особенности этой программы с которыми я столкнулся:_
* Введение в http пакет в go
* Обработка ошибок
*

![photo](https://github.com/Stepa-Nos/WebNote/blob/main/WebNote_photo1.png)
![photo](https://github.com/Stepa-Nos/WebNote/blob/main/WebNote_photo2.png)

--- 
## Сама программа  👾
```golang
package main

import (
	"io/ioutil"
	"log"
	"net/http"
	"regexp"
	"text/template"
)

type Page struct {
	Title string
	Body  []byte
}

var templates = template.Must(template.ParseFiles("edit.html", "view.html"))
var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)")

func (p *Page) save() error {
	fn := p.Title + ".txt"
	return ioutil.WriteFile(fn, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	fn := title + ".txt"
	body, err := ioutil.ReadFile(fn)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
	body := r.FormValue("body")
	p := Page{Title: title, Body: []byte(body)}
	err := p.save()
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	err := templates.ExecuteTemplate(w, tmpl+".html", p)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		m := validPath.FindStringSubmatch(r.URL.Path)
		if m == nil {
			http.NotFound(w, r)
			return
		}
		fn(w, r, m[2])
	}
}

func main() {
	http.HandleFunc("/view/", makeHandler(viewHandler))
	http.HandleFunc("/edit/", makeHandler(editHandler))
	http.HandleFunc("/save/", makeHandler(saveHandler))
	log.Fatal(http.ListenAndServe(":8088", nil))
}
