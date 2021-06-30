# The Simple Rest API Application

    package main

    import (
      "encoding/json"
      "log"
      "math/rand"
      "net/http"
      "strconv"

      "github.com/gorilla/mux"
    )

    // Book struct
    type Book struct {
      ID     string  `json:"id"`
      Isbn   string  `json:"isbn"`
      Title  string  `json:"title"`
      Author *Author `json:"author"`
    }

    // Author struct
    type Author struct {
      Firstname string `json:"firstname"`
      Lasttname string `json:"lastname"`
    }

    // Init books var as a slice Book struct
    var books []Book

    // Get All books
    func getBooks(w http.ResponseWriter, r *http.Request) {
      w.Header().Set("Content-Type", "Application/json")
      json.NewEncoder(w).Encode(books)
    }

    // Get Single book
    func getBook(w http.ResponseWriter, r *http.Request) {
      w.Header().Set("Content-Type", "Application/json")
      params := mux.Vars(r) // Get params
      // Loop through books and find with id
      for _, item := range books {
        if item.ID == params["id"] {
          json.NewEncoder(w).Encode(item)
          return
        }
      }
      json.NewEncoder(w).Encode(&Book{})
    }

    // Create a New Book
    func createBook(w http.ResponseWriter, r *http.Request) {
      w.Header().Set("Content-Type", "Application/json")
      var book Book
      _ = json.NewDecoder(r.Body).Decode(&book)
      book.ID = strconv.Itoa(rand.Intn(10000000)) //Mock ID
      books = append(books, book)
      json.NewEncoder(w).Encode(book)
    }

    //
    func updateBook(w http.ResponseWriter, r *http.Request) {
      w.Header().Set("Content-Type", "Application/json")
      params := mux.Vars(r)
      for index, item := range books {
        if item.ID == params["id"] {
          books = append(books[:index], books[index+1:]...)
          var book Book
          _ = json.NewDecoder(r.Body).Decode(&book)
          book.ID = params["id"] //Mock ID
          books = append(books, book)
          json.NewEncoder(w).Encode(book)
          return
        }
      }
      json.NewEncoder(w).Encode(books)
    }

    func deleteBook(w http.ResponseWriter, r *http.Request) {
      w.Header().Set("Content-Type", "Application/json")
      params := mux.Vars(r)
      for index, item := range books {
        if item.ID == params["id"] {
          books = append(books[:index], books[index+1:]...)
          break
        }
      }
      json.NewEncoder(w).Encode(books)
    }

    func main() {
      r := mux.NewRouter()

      // Mock Data - @todo - Implement DB
      books = append(books, Book{ID: "1", Isbn: "448743", Title: "Book One", Author: &Author{
        Firstname: "John",
        Lasttname: "Doe",
      }})

      books = append(books, Book{ID: "2", Isbn: "783265", Title: "Book Two", Author: &Author{
        Firstname: "Steve",
        Lasttname: "Smith",
      }})

      r.HandleFunc("/api/books", getBooks).Methods("GET")
      r.HandleFunc("/api/books/{id}", getBook).Methods("GET")
      r.HandleFunc("/api/books", createBook).Methods("POST")
      r.HandleFunc("/api/books/{id}", updateBook).Methods("PUT")
      r.HandleFunc("/api/books/{id}", deleteBook).Methods("DELETE")

      log.Fatal(http.ListenAndServe(":8000", r))
    }
