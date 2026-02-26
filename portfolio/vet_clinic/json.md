Para llevar este modelo a MongoDB de una forma óptima y clara, la mejor estrategia es usar un modelo mixto: anidar una entidad y referenciar la otra.

El esquema quedaría dividido en dos colecciones principales (`owners` y `appointments`):

```JSON
// ==========================================
// Collection: owners
// Represents the owner and their embedded pets
// ==========================================
{
  "_id": "ObjectId('65f2b3c...111')",
  "firstName": "Carlos",
  "lastName": "Gómez",
  "phone": "+522721234567",
  
  // Embedding the pets array directly into the owner's document (1-to-Few relationship).
  // This allows retrieving the owner and all their pets in a single database query.
  "pets": [
    {
      "petId": "ObjectId('65f2b3c...222')",
      "name": "Max",
      "species": "Dog",
      "breed": "Husky"
    },
    {
      "petId": "ObjectId('65f2b3c...333')",
      "name": "Luna",
      "species": "Cat",
      "breed": "Siamese"
    }
  ]
}

// ==========================================
// Collection: appointments
// Represents the scheduled visits, using references
// ==========================================
{
  "_id": "ObjectId('65f2b3c...999')",
  
  // Using references to link the appointment back to the specific pet and owner.
  // This prevents the owner's document from growing indefinitely over time.
  "petId": "ObjectId('65f2b3c...222')",
  "ownerId": "ObjectId('65f2b3c...111')",
  
  // Storing the appointment date as a native ISODate object for accurate range queries
  "appointmentDate": "2026-03-01T10:30:00.000Z",
  "reason": "Annual vaccination and general checkup"
}
```

### ¿Cuáles son embebidas y cuáles referenciadas?

**1. Mascotas (Entidad Embebida / Embedded)**

Las mascotas se guardan directamente dentro del documento del `Owner` como un arreglo de objetos.

- **¿Por qué?** Porque existe una relación **"Uno a Pocos" (1-to-Few)**. Una persona normalmente tiene de 1 a 5 mascotas, un número que no va a crecer descontroladamente. Al embeberlas, la base de datos es mucho más rápida, ya que con una sola consulta obtienes la información completa del cliente y sus animales sin necesidad de hacer un `$lookup` (JOIN).

**2. Citas (Entidad Referenciada / Referenced)**

Las citas se manejan en su propia colección separada y solo guardan el ID de la mascota (`petId`) y el ID del dueño (`ownerId`).

- **¿Por qué?** Porque existe una relación **"Uno a Muchos"** que puede ser **infinita**. Una mascota puede visitar la veterinaria cientos de veces a lo largo de su vida. Si anidáramos las citas dentro de la mascota, el documento del dueño crecería en cada visita, afectando el rendimiento y corriendo el riesgo de superar el límite de 16 MB de MongoDB.
- Además, tener las citas por separado es indispensable para la lógica del negocio: permite a la clínica consultar fácilmente "todas las citas de hoy", sin importar de quién sean las mascotas.

