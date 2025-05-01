# trial<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Books</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet"/>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet"/>
  <style>
    th[data-sortable] {
      cursor: pointer;
    }
    th[data-sortable]:hover {
      background-color: #f8f9fa;
    }
  </style>
</head>
<body class="container my-5">
  <h2 class="mb-4">Book Master</h2>
  
  <form id="bookForm" class="mb-4">
    <div class="col-md-4">
      <label for="BookID" class="form-label">Book Tile</label>
      <select id="BookID" class="form-select">
        <option value="">-- Select --</option>
      </select>
      <div class="invalid-feedback">Please select an Book.</div>
    </div>
    <!-- Author Dropdown -->
  <div class="col-md-4">
    <label for="AuthorID" class="form-label">Author</label>
    <select id="AuthorID" class="form-select">
      <option value="">-- Select Author --</option>
    </select>
    <div class="invalid-feedback">Please select an Author.</div>
  </div>
  <div class="mb-3">
      <label for="genre" class="form-label">Genre</label>
      <input type="text" class="form-control" id="genre" required>
      <div class="invalid-feedback">At least 5 characters required.</div>
    </div>
    <div class="mb-3">
      <label for="totalCopies" class="form-label">Total Copies</label>
      <input type="number" class="form-control" id="totalCopies" required>
    </div>
    <div class="mb-3">
      <label for="availableCopies" class="form-label">Available Copies</label>
      <input type="number" class="form-control" id="availableCopies" required>
    </div>
    <button type="submit" class="btn btn-primary" id="submitBtn">Add Book</button>
  </form>

  <div class="mb-3">
    <input type="text" class="form-control" id="searchInput" placeholder="Search by Book Title...">
  </div>

  <table class="table table-bordered">
    <thead>
      <tr>
        <th>ID</th>
        <th data-sortable="true">Title <i class="bi bi-arrow-down-up"></i></th>
        <th>Author ID</th>
        <th>Genre</th>
        <th>Total</th>
        <th>Available</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody id="bookTableBody"></tbody>
  </table>

  <script>
    const apiUrl = "http://localhost:3000/Books";
    const form = document.getElementById("bookForm");
    const titleInput = document.getElementById("title");
    const authorIdInput = document.getElementById("authorId");
    const genreInput = document.getElementById("genre");
    const totalCopiesInput = document.getElementById("totalCopies");
    const availableCopiesInput = document.getElementById("availableCopies");
    const submitBtn = document.getElementById("submitBtn");
    const searchInput = document.getElementById("searchInput");
    const bookTableBody = document.getElementById("bookTableBody");
    const bookSelect = document.getElementById("BookID"); // Dropdown for Book Titles
    const authorDropdown = document.getElementById("AuthorID"); // Author dropdown


    let editId = null;
    let allBooks = [];
    let sortDirection = 1;
    let currentSortField = null;

    document.addEventListener("DOMContentLoaded", function() {
      fetchAndRenderBooks();
      loadBooks();
      loadAuthors();
      setupEventListeners();
    });

    // Function to load authors
  function loadAuthors() {
    fetch("http://localhost:8080/api/authors") // Adjust URL for your API
      .then(res => res.json())
      .then(data => {
        authorDropdown.innerHTML = '<option value="">-- Select Author --</option>';
        data.forEach(author => {
          const option = document.createElement("option");
          option.value = author.id; // assuming each author has an id and name
          option.textContent = author.name;
          authorDropdown.appendChild(option);
        });
      })
      .catch(error => {
        console.error("Error fetching authors:", error);
        alert("Failed to fetch authors.");
      });
  }


    function fetchAndRenderBooks() {
      if (apiUrl.includes("localhost")) {
        allBooks = [
          {
            BookID: 1,
            Title: "Harry Potter",
            AuthorID: 1,
            Genre: "Fantasy",
            TotalCopies: 10,
            AvailableCopies: 5,
            id: "9d11"
          }
        ];
        renderBooks(allBooks);
      } else {
        fetch(apiUrl)
          .then(res => res.json())
          .then(data => {
            allBooks = data;
            renderBooks(data);
          })
          .catch(error => {
            console.error("Error fetching books:", error);
            alert("Failed to fetch books. See console for details.");
          });
      }
    }
    // Modification I made
    function loadBooks() {
      // Function to populate the Book Title dropdown dynamically
      fetch(apiUrl)
        .then((res) => res.json())
        .then((data) => {
          // Populate dropdown with Book Titles
          data.forEach((book) => {
            const option = document.createElement("option");
            option.value = book.BookID;
            option.textContent = book.Title; // Display the Book Title
            bookSelect.appendChild(option);
          });
        })
      }
    function renderBooks(books) {
      bookTableBody.innerHTML = "";

      if (books.length === 0) {
        const row = document.createElement("tr");
        row.innerHTML = `<td colspan="7" class="text-center">No books found</td>`;
        bookTableBody.appendChild(row);
        return;
      }

      books.forEach(book => {
        const row = document.createElement("tr");
        row.innerHTML = `
          <td>${book.BookID}</td>
          <td>${book.Title}</td>
          <td>${book.AuthorID}</td>
          <td>${book.Genre}</td>
          <td>${book.TotalCopies}</td>
          <td>${book.AvailableCopies}</td>
          <td>
            <button class="btn btn-sm btn-outline-primary me-1 edit-btn" data-id="${book.id}">
              <i class="bi bi-pencil-square"></i>
            </button>
            <button class="btn btn-sm btn-outline-danger delete-btn" data-id="${book.id}">
              <i class="bi bi-trash"></i>
            </button>
          </td>
        `;
        bookTableBody.appendChild(row);
      });

      document.querySelectorAll(".edit-btn").forEach(btn => {
        btn.addEventListener("click", () => {
          const id = btn.getAttribute("data-id");
          loadBookForEdit(id);
        });
      });

      document.querySelectorAll(".delete-btn").forEach(btn => {
        btn.addEventListener("click", () => {
          const id = btn.getAttribute("data-id");
          deleteBook(id);
        });
      });
    }

    function deleteBook(id) {
      if (confirm("Are you sure you want to delete this book?")) {
        if (apiUrl.includes("localhost")) {
          allBooks = allBooks.filter(book => book.id !== id);
          renderBooks(allBooks);
          alert("Book deleted (mock operation - no actual API call)");
        } else {
          fetch(`${apiUrl}/${id}`, {
            method: "DELETE"
          })
          .then(() => fetchAndRenderBooks())
          .catch(error => {
            console.error("Error deleting book:", error);
            alert("Failed to delete book. See console for details.");
          });
        }
      }
    }

    function loadBookForEdit(id) {
      const book = allBooks.find(b => b.id === id);
      if (book) {
        titleInput.value = book.Title;
        authorIdInput.value = book.AuthorID;
        genreInput.value = book.Genre;
        totalCopiesInput.value = book.TotalCopies;
        availableCopiesInput.value = book.AvailableCopies;
        editId = id;
        submitBtn.textContent = "Update Book";
        scrollTo(0, 0);
      }
    }

    function loadAuthors() {
      //alert("Loading Authors");
      fetch(`${apiUrl}/Books`)
        .then((res) => res.json())
        .then((data) => {
          books = data;
          const bookSelect = document.getElementById("BookID");
          data.forEach((au) => {
            bookSelect.innerHTML += `<option value="${au.BookID}">${au.Name}</option>`;
          });
        });
    }

    function setupEventListeners() {
      form.addEventListener("submit", function(e) {
        e.preventDefault();
        if (!validateForm()) return;

        const book = {
          Title: titleInput.value.trim(),
          AuthorID: parseInt(authorIdInput.value),
          Genre: genreInput.value.trim(),
          TotalCopies: parseInt(totalCopiesInput.value),
          AvailableCopies: parseInt(availableCopiesInput.value)
        };

        if (editId) {
          if (apiUrl.includes("localhost")) {
            const index = allBooks.findIndex(b => b.id === editId);
            if (index !== -1) {
              allBooks[index] = { ...allBooks[index], ...book };
              renderBooks(allBooks);
              resetForm();
              alert("Book updated (mock operation - no actual API call)");
            }
          } else {
            fetch(`${apiUrl}/${editId}`, {
              method: "PUT",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify(book)
            })
            .then(() => {
              fetchAndRenderBooks();
              resetForm();
            })
            .catch(error => {
              console.error("Error updating book:", error);
              alert("Failed to update book. See console for details.");
            });
          }
        } else {
          if (apiUrl.includes("localhost")) {
            const newId = allBooks.length > 0 ? Math.max(...allBooks.map(b => b.BookID)) + 1 : 1;
            const newBook = { ...book, BookID: newId, id: Math.random().toString(36).substring(2, 7) };
            allBooks.push(newBook);
            renderBooks(allBooks);
            resetForm();
            alert("Book added (mock operation - no actual API call)");
          } else {
            fetch(apiUrl, {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify(book)
            })
            .then(() => {
              fetchAndRenderBooks();
              resetForm();
            })
            .catch(error => {
              console.error("Error adding book:", error);
              alert("Failed to add book. See console for details.");
            });
          }
        }
      });

      searchInput.addEventListener("input", () => {
        const searchTerm = searchInput.value.toLowerCase();
        const filtered = allBooks.filter(book =>
          book.Title.toLowerCase().includes(searchTerm) ||
          book.Genre.toLowerCase().includes(searchTerm)
        );
        renderBooks(filtered);
      });

      document.querySelectorAll("th[data-sortable]").forEach(header => {
        header.addEventListener("click", () => {
          const field = header.textContent.trim();
          sortBooks(field);
        });
      });

      [titleInput, genreInput].forEach(input => {
        input.addEventListener("blur", () => validateInput(input));
        input.addEventListener("input", () => {
          if (input.classList.contains("is-invalid")) {
            validateInput(input);
          }
        });
      });
    }

    function sortBooks(field) {
      if (currentSortField === field) {
        sortDirection *= -1;
      } else {
        currentSortField = field;
        sortDirection = 1;
      }

      const sorted = [...allBooks].sort((a, b) => {
        const aValue = a[field] || "";
        const bValue = b[field] || "";
        return aValue.toString().localeCompare(bValue.toString()) * sortDirection;
      });

      renderBooks(sorted);
    }

    function validateInput(input) {
      if (input.value.trim().length < 5) {
        input.classList.add("is-invalid");
        input.classList.remove("is-valid");
      } else {
        input.classList.remove("is-invalid");
        input.classList.add("is-valid");
      }
    }

    function validateForm() {
      let isValid = true;
      [titleInput, genreInput].forEach(input => {
        if (input.value.trim().length < 5) {
          input.classList.add("is-invalid");
          input.classList.remove("is-valid");
          isValid = false;
        } else {
          input.classList.remove("is-invalid");
          input.classList.add("is-valid");
        }
      });
      return isValid;
    }

    function resetForm() {
      form.reset();
      editId = null;
      submitBtn.textContent = "Add Book";
      [titleInput, genreInput].forEach(input => {
        input.classList.remove("is-valid", "is-invalid");
      });
    }
  </script>
</body>
</html>
