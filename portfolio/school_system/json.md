## MongoDB schema

Este es el punto exacto donde el diseño de bases de datos NoSQL difiere radicalmente del modelo relacional tradicional.

Para explicarlo forma clara, vamos a dividir este modelo en tres colecciones (collections). Mantenemos todas las llaves (keys), valores de ejemplo y los comentarios explicativos en inglés dentro de los esquemas.

### 1. Colección de Estudiantes (`students`)

Aquí vamos a resolver tres de las relaciones: **Uno a Uno**, **Uno a Pocos** y parte del **Muchos a Muchos**.

```JSON
// ==========================================
// Collection: students
// Demonstrates 1:1, 1:Few (Embedded) and N:M (Referenced)
// ==========================================
{
  "_id": "ObjectId('65f3c1...001')",
  "firstName": "Ana",
  "lastName": "López",
  "email": "ana.lopez@example.com",

  // 1:1 RELATIONSHIP (One-to-One) - EMBEDDED
  // A student has exactly one medical profile. Since it's a fixed size 
  // and accessed together with the student data, embedding is the best choice.
  "profile": {
    "bloodType": "O+",
    "medicalConditions": "None"
  },

  // 1:Few RELATIONSHIP (One-to-Few) - EMBEDDED
  // A student typically has 1 to 3 emergency contacts. This array will 
  // not grow indefinitely, making it safe and efficient to embed.
  "emergencyContacts": [
    {
      "fullName": "Carlos López",
      "relationship": "Father",
      "phoneNumber": "555-0101"
    },
    {
      "fullName": "María García",
      "relationship": "Mother",
      "phoneNumber": "555-0102"
    }
  ],

  // N:M RELATIONSHIP (Many-to-Many) - REFERENCED
  // A student takes multiple courses. We store an array of references (ObjectIds) 
  // pointing to the 'courses' collection. 
  "enrolledCourses": [
    "ObjectId('65f3c1...C01')",
    "ObjectId('65f3c1...C02')"
  ]
}
```

### 2. Colección de Asistencia (`attendance_records`)

Aquí resolvemos la relación **Uno a Muchos**, que en MongoDB se maneja de forma distinta a la de "Uno a Pocos" por el factor de crecimiento.

```JSON
// ==========================================
// Collection: attendance_records
// Demonstrates 1:N (One-to-Many / Unbounded) - REFERENCED
// ==========================================
{
  "_id": "ObjectId('65f3c1...A01')",
  
  // We use a reference to the student instead of embedding the attendance 
  // array inside the student's document.
  // WHY?: A student generates a new attendance record every class. If embedded,
  // the array would grow infinitely (Unbounded), slowing down queries and 
  // eventually hitting MongoDB's 16MB document limit.
  "studentId": "ObjectId('65f3c1...001')",
  
  "classDate": "2026-02-26T08:00:00.000Z",
  "status": "Present"
}
```

### 3. Colección de Cursos (`courses`)

Esta es la otra mitad de la relación **Muchos a Muchos**.

```JSON
// ==========================================
// Collection: courses
// Part of the N:M relationship
// ==========================================
{
  "_id": "ObjectId('65f3c1...C01')",
  "courseName": "Database Systems",
  "credits": 5
  
  // Notice we don't necessarily need an array of studentIds here if we already 
  // have the enrolledCourses array in the student document. In MongoDB, we 
  // optimize for the most common query (e.g., "What courses is Ana taking?").
}
```

### Puntos clave para discutir en clase:

- **La regla de oro del tamaño:** Siempre piensen en el límite de los 16 MB. Si el arreglo va a crecer cada día (como la asistencia), **se referencia**. Si el arreglo tiene un límite natural pequeño (como los números de emergencia de los papás), **se embebe**.
- **Velocidad vs. Normalización:** Embeber datos (1:1 y 1:Pocos) es como tener todo en una sola hoja de papel; es rapidísimo de leer. Referenciar datos requiere que la base de datos "busque en otra hoja" (usando `$lookup`), lo cual es un poco más lento pero mantiene la información organizada y evita que el documento explote en tamaño.
- **Muchos a Muchos direccional:** En el caso de los cursos, guardamos los IDs de las materias dentro del estudiante. *"¿Qué pasaría si también guardáramos un arreglo con miles de IDs de estudiantes dentro de cada curso?"*. La respuesta es que también podría volverse un arreglo gigante y difícil de mantener, por lo que a menudo se prefiere mantener la referencia solo de un lado, dependiendo de qué consulta se haga con más frecuencia en la aplicación.
