package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
	"strconv"
)

type Usuario struct {
	ID     int    `json:"id"`
	Nombre string `json:"name"`
	Email  string `json:"email"`
}

var usuarios = []Usuario{
	{ID: 1, Nombre: "Antonio", Email: "Antonio15@gmail.com"},
}

// Obtener todos los usuarios
func obtenerUsuarios(c *gin.Context) {
	c.JSON(http.StatusOK, usuarios)
}

// Crear nuevo usuario
func crearUsuario(c *gin.Context) {
	var nuevo Usuario
	if err := c.ShouldBindJSON(&nuevo); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "JSON inválido"})
		return
	}
	nuevo.ID = len(usuarios) + 1
	usuarios = append(usuarios, nuevo)
	c.JSON(http.StatusCreated, nuevo)
}

// Actualizar usuario por ID
func actualizarUsuario(c *gin.Context) {
	idParam := c.Param("id")
	id, err := strconv.Atoi(idParam)
	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "ID inválido"})
		return
	}

	var datosActualizados Usuario
	if err := c.ShouldBindJSON(&datosActualizados); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "JSON inválido"})
		return
	}

	for i, u := range usuarios {
		if u.ID == id {
			usuarios[i].Nombre = datosActualizados.Nombre
			usuarios[i].Email = datosActualizados.Email
			c.JSON(http.StatusOK, usuarios[i])
			return
		}
	}
	c.JSON(http.StatusNotFound, gin.H{"error": "Usuario no encontrado"})
}

// Eliminar usuario por ID
func eliminarUsuario(c *gin.Context) {
	idParam := c.Param("id")
	id, err := strconv.Atoi(idParam)
	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "ID inválido"})
		return
	}

	for i, u := range usuarios {
		if u.ID == id {
			usuarios = append(usuarios[:i], usuarios[i+1:]...)
			c.Status(http.StatusNoContent)
			return
		}
	}
	c.JSON(http.StatusNotFound, gin.H{"error": "Usuario no encontrado"})
}

// Ruta de prueba
func ping(c *gin.Context) {
	c.String(http.StatusOK, "pong")
}

func main() {
	r := gin.Default()

	r.GET("/ping", ping)

	r.GET("/v1/users", obtenerUsuarios)
	r.POST("/v1/users", crearUsuario)
	r.PUT("/v1/users/:id", actualizarUsuario)     // usando URL param
	r.DELETE("/v1/users/:id", eliminarUsuario)    // usando URL param

	r.Run(":3000")
}
