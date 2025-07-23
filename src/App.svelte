<script>
  import axios from 'axios';
  import { onMount } from 'svelte';

  let vista = 'estudiantes';

  // Datos
  let estudiantes = [];
  let profesores = [];
  let cursos = [];
  let evaluaciones = [];
  let matriculas = [];


  // Formulario Estudiante
let nuevoEstudiante = {
  nombre: '',
  apellido: '',
  correo: '',
  fecha_nacimiento: '',
  carrera: ''
};

// Formulario Profesor
let nuevoProfesor = {
  nombre: '',
  apellido: '',
  correo: '',
  especialidad: ''
};

// Formulario Curso
let nuevoCurso = {
  nombre: '',
  codigo: '',
  creditos: '',
  id_profesor: ''
};

// Formulario Evaluación
let nuevaEvaluacion = {
  id_matricula: '',
  tipo: '',
  nota: '',
  fecha: ''
};

// Estados de modal
let modalEntidad = ''; // estudiante, profesor, curso, evaluacion

  // Modal y formulario
  let mostrarModal = false;
  let nuevaMatricula = {
    id_estudiante: '',
    id_curso: '',
    semestre: ''
  };

  // Carga dinámica de datos
  onMount(async () => {
    try {
      const e = await axios.get('/api/estudiantes');
      estudiantes = e.data;

      const p = await axios.get('/api/profesores');
      profesores = p.data;

      const c = await axios.get('/api/cursos');
      cursos = c.data;

      const evalRes = await axios.get('/api/evaluaciones');
      evaluaciones = evalRes.data;

      const matRes = await axios.get('/api/matriculas');
      matriculas = matRes.data;

    } catch (err) {
      console.error('Error al cargar datos:', err);
    }
  });

  function formatearFecha(fecha) {
  if (!fecha) return '';
  return fecha.split('T')[0]; // "1999-10-31"
}

function editarEstudiante(est) {
  nuevoEstudiante = {
    ...est,
    fecha_nacimiento: formatearFecha(est.fecha_nacimiento)
  };
  modalEntidad = 'estudiante';
}

async function eliminarEstudiante(id) {
  if (confirm('¿Desea eliminar este estudiante?')) {
    await axios.delete(`/api/estudiantes/${id}`);
    const res = await axios.get('/api/estudiantes');
    estudiantes = res.data.estudiante;
  }
}

function editarProfesor(prof) {
  nuevoProfesor = { ...prof };
  modalEntidad = 'profesor';
}

async function eliminarProfesor(id) {
  if (confirm('¿Desea eliminar este profesor?')) {
    await axios.delete(`/api/profesores/${id}`);
    const res = await axios.get('/api/profesores');
    profesores = res.data.profesor;
  }
}



  async function registrarEntidad(tipo) {
  try {
    if (tipo === 'estudiante') {
      await axios.post('/api/estudiantes', nuevoEstudiante);
      const res = await axios.get('/api/estudiantes');
      estudiantes = res.data.estudiante;
      nuevoEstudiante = { nombre: '', apellido: '', correo: '', fecha_nacimiento: '', carrera: '' };
    }
    if (tipo === 'profesor') {
      await axios.post('/api/profesores', nuevoProfesor);
      const res = await axios.get('/api/profesores');
      profesores = res.data.profesor;
      nuevoProfesor = { nombre: '', apellido: '', correo: '', especialidad: '' };
    }
    if (tipo === 'curso') {
      await axios.post('/api/cursos', nuevoCurso);
      const res = await axios.get('/api/cursos');
      cursos = res.data.curso;
      nuevoCurso = { nombre: '', codigo: '', creditos: '', id_profesor: '' };
    }
    if (tipo === 'evaluacion') {
  if (nuevaEvaluacion.id_evaluacion) {
    // Actualizar
    await axios.put(`/api/evaluaciones/${nuevaEvaluacion.id_evaluacion}`, nuevaEvaluacion);
  } else {
    // Crear nueva
    await axios.post('/api/evaluaciones', nuevaEvaluacion);
  }

  const res = await axios.get('/api/evaluaciones');
  evaluaciones = res.data.evaluacion;

  nuevaEvaluacion = { id_matricula: '', tipo: '', nota: '', fecha: '', id_evaluacion: '' };
}

    modalEntidad = '';
  } catch (err) {
    console.error('Error al registrar:', err);
    alert('Error al registrar');
  }
}

  // Funciones auxiliares
  function obtenerMatricula(id_matricula) {
    return matriculas.find(m => m.id_matricula === id_matricula);
  }

  function obtenerEstudianteNombre(id_estudiante) {
    const e = estudiantes.find(e => e.id_estudiante === id_estudiante);
    return e ? `${e.nombre} ${e.apellido}` : 'Desconocido';
  }

  function obtenerNombreCurso(id_curso) {
    const c = cursos.find(c => c.id_curso === id_curso);
    return c ? c.nombre : 'Desconocido';
  }

  function obtenerNombreProfesor(id) {
    const prof = profesores.find(p => p.id_profesor === id);
    return prof ? `${prof.nombre} ${prof.apellido}` : 'No asignado';
  }

  async function registrarMatricula() {
    try {
      await axios.post('/api/matriculas', {
        id_estudiante: nuevaMatricula.id_estudiante,
        id_curso: nuevaMatricula.id_curso,
        semestre: nuevaMatricula.semestre,
        fecha_matricula: new Date().toISOString().split('T')[0]
      });

      alert('Matrícula registrada correctamente');
      mostrarModal = false;
      nuevaMatricula = { id_estudiante: '', id_curso: '', semestre: '' };

      const matRes = await axios.get('/api/matriculas');
      matriculas = matRes.data.matricula;
    } catch (error) {
      console.error('Error al registrar matrícula', error);
      alert('Hubo un error al registrar la matrícula.');
    }
  }

  function editarCurso(curso) {
  nuevoCurso = { ...curso };
  modalEntidad = 'curso';
}

async function eliminarCurso(id) {
  if (confirm('¿Desea eliminar este curso?')) {
    await axios.delete(`/api/cursos/${id}`);
    const res = await axios.get('/api/cursos');
    cursos = res.data.curso;
  }
}

function editarMatricula(m) {
  nuevaMatricula = {
    id_estudiante: m.id_estudiante,
    id_curso: m.id_curso,
    semestre: m.semestre,
    id_matricula: m.id_matricula // opcional para PUT
  };
  modalEntidad = 'matricula';
}

async function eliminarMatricula(id) {
  if (confirm('¿Desea eliminar esta matrícula?')) {
    await axios.delete(`/api/matriculas/${id}`);
    const matRes = await axios.get('/api/matriculas');
    matriculas = matRes.data.matricula;
  }
}

function editarEvaluacion(ev) {
  nuevaEvaluacion = {
    id_evaluacion: ev.id_evaluacion, // para PUT
    id_matricula: ev.id_matricula,
    tipo: ev.tipo,
    nota: ev.nota,
    fecha: ev.fecha
  };
  modalEntidad = 'evaluacion';
}

async function eliminarEvaluacion(id) {
  if (confirm('¿Eliminar esta evaluación?')) {
    await axios.delete(`/api/evaluaciones/${id}`);
    const res = await axios.get('/api/evaluaciones');
    evaluaciones = res.data.evaluacion;
  }
}



</script>

<main class="min-h-screen w-full p-4">
  <h1 class="text-xl font-bold mb-2">📚 Sistema Académico</h1>
  <!-- Navegación -->
  <ul class="nav nav-tabs mb-3">
    <li class="nav-item">
      <button type="button" class="nav-link btn btn-link {vista === 'estudiantes' ? 'active' : ''}" on:click={() => vista = 'estudiantes'}>
        Estudiantes
      </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link btn btn-link {vista === 'profesores' ? 'active' : ''}" on:click={() => vista = 'profesores'}>
        Profesores
      </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link btn btn-link {vista === 'cursos' ? 'active' : ''}" on:click={() => vista = 'cursos'}>
        Cursos
      </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link btn btn-link {vista === 'matriculas' ? 'active' : ''}" on:click={() => vista = 'matriculas'}>
        Matrículas
      </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link btn btn-link {vista === 'evaluaciones' ? 'active' : ''}" on:click={() => vista = 'evaluaciones'}>
        Evaluaciones
      </button>
    </li>
  </ul>

  <!-- Estudiantes -->
{#if vista === 'estudiantes'}
  <div class="flex justify-between items-center mb-4">
    <button
      class="bg-gray-100 text-white/90 px-5 py-2 rounded shadow-sm hover:bg-gray-200 transition duration-200 flex items-center gap-2 border border-gray-300"
      on:click={() => modalEntidad = 'estudiante'}
    >
      <span class="text-xl" style="filter: drop-shadow(0 0 1px #8b5cf6); color: #8b5cf6;">➕</span>
      Nuevo Estudiante
    </button>
  </div>

  <div class="w-full overflow-x-auto shadow rounded">
    <table class="w-full table-fixed text-sm text-left border border-gray-200">
      <thead class="bg-gray-100 text-gray-700">
        <tr>
          <th class="px-4 py-2 w-[5%]">ID</th>
          <th class="px-4 py-2 w-[15%]">Nombre</th>
          <th class="px-4 py-2 w-[15%]">Apellido</th>
          <th class="px-4 py-2 w-[25%]">Correo</th>
          <th class="px-4 py-2 w-[15%]">Nacimiento</th>
          <th class="px-4 py-2 w-[15%]">Carrera</th>
          <th class="px-4 py-2 w-[10%]">Acciones</th>
        </tr>
      </thead>
      <tbody class="divide-y divide-gray-200">
        {#each estudiantes as e}
          <tr>
            <td class="px-4 py-2 w-[5%]">{e.id_estudiante}</td>
            <td class="px-4 py-2 w-[15%]">{e.nombre}</td>
            <td class="px-4 py-2 w-[15%]">{e.apellido}</td>
            <td class="px-4 py-2 w-[25%]">{e.correo}</td>
            <td class="px-4 py-2 w-[15%]">{formatearFecha(e.fecha_nacimiento)}</td>
            <td class="px-4 py-2 w-[15%]">{e.carrera}</td>
            <td class="px-4 py-2 w-[10%] flex gap-2">
              <button
                class="bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600"
                on:click={() => editarEstudiante(e)}
                aria-label="Editar"
              >
                ✏️
              </button>
              <button
                class="bg-red-600 text-white px-2 py-1 rounded hover:bg-red-700"
                on:click={() => eliminarEstudiante(e.id_estudiante)}
                aria-label="Eliminar"
              >
                🗑️
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
{/if}

  <!-- Profesores -->
  {#if vista === 'profesores'}
    <div class="flex justify-between items-center mb-4">
  <button
  class="bg-gray-100 text-white/90 px-5 py-2 rounded shadow-sm hover:bg-gray-200 transition duration-200 flex items-center gap-2 border border-gray-300"
  on:click={() => modalEntidad = 'profesor'}
>
  <span class="text-xl" style="filter: drop-shadow(0 0 1px #8b5cf6); color: #8b5cf6;">➕</span>
  Nuevo Profesor
</button>
</div>

<div class="w-fuloverflow-x-auto shadow rounded w-full">
  <table class="w-full table-fixed text-sm text-left border border-gray-200">
    <thead class="bg-gray-100 text-gray-700">
      <tr>
        <th class="px-4 py-2">ID</th>
        <th class="px-4 py-2">Nombre</th>
        <th class="px-4 py-2">Apellido</th>
        <th class="px-4 py-2">Correo</th>
        <th class="px-4 py-2">Especialidad</th>
        <th class="px-4 py-2">Acciones</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-gray-200">
      {#each profesores as p}
        <tr>
          <td class="px-4 py-2">{p.id_profesor}</td>
          <td class="px-4 py-2">{p.nombre}</td>
          <td class="px-4 py-2">{p.apellido}</td>
          <td class="px-4 py-2">{p.correo}</td>
          <td class="px-4 py-2">{p.especialidad}</td>
           <td class="px-4 py-2 w-[10%] flex gap-2">
              <button
                class="bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600"
                on:click={() => editarProfesor(p)}
                aria-label="Editar"
              >
                ✏️
              </button>
              <button
                class="bg-red-600 text-white px-2 py-1 rounded hover:bg-red-700"
                on:click={() => eliminarProfesor(p.id_profesor)}
                aria-label="Eliminar"
              >
                🗑️
              </button>
            </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

  {/if}

  <!-- Cursos -->
  {#if vista === 'cursos'}
    <div class="flex justify-between items-center mb-4">
  <button
  class="bg-gray-100 text-white/90 px-5 py-2 rounded shadow-sm hover:bg-gray-200 transition duration-200 flex items-center gap-2 border border-gray-300"
  on:click={() => modalEntidad = 'curso'}
>
  <span class="text-xl" style="filter: drop-shadow(0 0 1px #8b5cf6); color: #8b5cf6;">➕</span>
  Nuevo Curso
</button>
</div>

<div class="w-fuloverflow-x-auto shadow rounded w-full">
  <table class="w-full text-sm text-left table-auto border border-gray-200">
    <thead class="bg-gray-100 text-gray-700">
      <tr>
        <th class="px-4 py-2">ID</th>
        <th class="px-4 py-2">Código</th>
        <th class="px-4 py-2">Nombre</th>
        <th class="px-4 py-2">Créditos</th>
        <th class="px-4 py-2">Profesor</th>
        <th class="px-4 py-2">Acciones</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-gray-200">
      {#each cursos as c}
        <tr>
          <td class="px-4 py-2">{c.id_curso}</td>
          <td class="px-4 py-2">{c.codigo}</td>
          <td class="px-4 py-2">{c.nombre}</td>
          <td class="px-4 py-2">{c.creditos}</td>
          <td class="px-4 py-2">{obtenerNombreProfesor(c.id_profesor)}</td>
           <td class="px-4 py-2 w-[10%] flex gap-2">
              <button
                class="bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600"
                on:click={() => editarCurso(c)}
                aria-label="Editar"
              >
                ✏️
              </button>
              <button
                class="bg-red-600 text-white px-2 py-1 rounded hover:bg-red-700"
                on:click={() => eliminarCurso(c.id_curso)}
                aria-label="Eliminar"
              >
                🗑️
              </button>
            </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

    <!-- Botón y Modal -->

    {#if mostrarModal}
      <div class="modal fade show d-block" tabindex="-1" style="background-color: rgba(0,0,0,0.5);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">Registrar Matrícula</h5>
              <button type="button" class="btn-close" aria-label="Cerrar" on:click={() => mostrarModal = false}></button>
            </div>
            <div class="modal-body">
              <div class="mb-3">
                <label>Estudiante</label>
                <select class="form-control" bind:value={nuevaMatricula.id_estudiante}>
                  <option value="">Seleccione un estudiante</option>
                  {#each estudiantes as e}
                    <option value={e.id_estudiante}>{e.nombre} {e.apellido}</option>
                  {/each}
                </select>
              </div>
              <div class="mb-3">
                <label>Curso</label>
                <select class="form-control" bind:value={nuevaMatricula.id_curso}>
                  <option value="">Seleccione un curso</option>
                  {#each cursos as c}
                    <option value={c.id_curso}>{c.nombre}</option>
                  {/each}
                </select>
              </div>
              <div class="mb-3">
                <label>Semestre</label>
                <input class="form-control" bind:value={nuevaMatricula.semestre} placeholder="Ej: 9" />
              </div>
            </div>
            <div class="modal-footer">
              <button class="btn btn-secondary" on:click={() => mostrarModal = false}>Cancelar</button>
              <button class="btn btn-success" on:click={registrarMatricula}>Guardar</button>
            </div>
          </div>
        </div>
      </div>
    {/if}
  {/if}

  <!-- Evaluaciones -->
{#if vista === 'evaluaciones'}
  <div class="flex justify-between items-center mb-4">
  <button
  class="bg-gray-100 text-white/90 px-5 py-2 rounded shadow-sm hover:bg-gray-200 transition duration-200 flex items-center gap-2 border border-gray-300"
  on:click={() => modalEntidad = 'evaluacion'}
>
  <span class="text-xl" style="filter: drop-shadow(0 0 1px #8b5cf6); color: #8b5cf6;">➕</span>
  Nuevo Evaluación
</button>
</div>

<div class="w-fuloverflow-x-auto shadow rounded w-full">
  <table class="w-full text-sm text-left table-auto border border-gray-200">
    <thead class="bg-gray-100 text-gray-700">
      <tr>
        <th class="px-4 py-2">ID</th>
        <th class="px-4 py-2">Tipo</th>
        <th class="px-4 py-2">Nota</th>
        <th class="px-4 py-2">Fecha</th>
        <th class="px-4 py-2">Estudiante</th>
        <th class="px-4 py-2">Curso</th>
        <th class="px-4 py-2">Acciones</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-gray-200">
      {#each evaluaciones as ev}
        <tr>
          <td class="px-4 py-2">{ev.id_evaluacion}</td>
          <td class="px-4 py-2">{ev.tipo}</td>
          <td class="px-4 py-2">{ev.nota}</td>
          <td class="px-4 py-2">{formatearFecha(ev.fecha)}</td>
          <td class="px-4 py-2">
            {obtenerEstudianteNombre(obtenerMatricula(ev.id_matricula)?.id_estudiante)}
          </td>
          <td class="px-4 py-2">
            {obtenerNombreCurso(obtenerMatricula(ev.id_matricula)?.id_curso)}
          </td>
           <td class="px-4 py-2 w-[10%] flex gap-2">
              <button
                class="bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600"
                on:click={() => editarEvaluacion(ev)}
                aria-label="Editar"
              >
                ✏️
              </button>
              <button
                class="bg-red-600 text-white px-2 py-1 rounded hover:bg-red-700"
                on:click={() => eliminarEvaluacion(ev.id_evaluacion)}
                aria-label="Eliminar"
              >
                🗑️
              </button>
            </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

{/if}

<!-- Vista de Matrículas -->
{#if vista === 'matriculas'}
  <div class="flex justify-between items-center mb-4">
  <button
  class="bg-gray-100 text-white/90 px-5 py-2 rounded shadow-sm hover:bg-gray-200 transition duration-200 flex items-center gap-2 border border-gray-300"
  on:click={() => modalEntidad = 'matricula'}
>
  <span class="text-xl" style="filter: drop-shadow(0 0 1px #8b5cf6); color: #8b5cf6;">➕</span>
  Nueva Matrícula
</button>
</div>

<div class="w-fuloverflow-x-auto shadow rounded w-full">
  <table class="w-full table-fixed text-sm text-left border border-gray-200">
    <thead class="bg-gray-100 text-gray-700">
      <tr>
        <th class="px-4 py-2">ID</th>
        <th class="px-4 py-2">Estudiante</th>
        <th class="px-4 py-2">Curso</th>
        <th class="px-4 py-2">Semestre</th>
        <th class="px-4 py-2">Fecha</th>
        <th class="px-4 py-2">Acciones</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-gray-200">
      {#each matriculas as m}
        <tr>
          <td class="px-4 py-2">{m.id_matricula}</td>
          <td class="px-4 py-2">{obtenerEstudianteNombre(m.id_estudiante)}</td>
          <td class="px-4 py-2">{obtenerNombreCurso(m.id_curso)}</td>
          <td class="px-4 py-2">{m.semestre}</td>
          <td class="px-4 py-2">{formatearFecha(m.fecha_matricula)}</td>
           <td class="px-4 py-2 w-[10%] flex gap-2">
              <button
                class="bg-yellow-500 text-white px-2 py-1 rounded hover:bg-yellow-600"
                on:click={() => editarMatricula(m)}
                aria-label="Editar"
              >
                ✏️
              </button>
              <button
                class="bg-red-600 text-white px-2 py-1 rounded hover:bg-red-700"
                on:click={() => eliminarMatricula(m.id_matricula)}
                aria-label="Eliminar"
              >
                🗑️
              </button>
            </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

{/if}



{#if modalEntidad}
  <div class="modal fade show d-block" tabindex="-1" style="background: rgba(0,0,0,0.5);">
    <div class="modal-dialog modal-lg">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Registrar {modalEntidad}</h5>
          <button class="btn-close" aria-label="Cerrar" on:click={() => modalEntidad = ''}></button>
        </div>
        <div class="modal-body">
          {#if modalEntidad === 'estudiante'}
            <label for="nombre">Nombre</label>
            <input id="nombre" class="form-control mb-2" bind:value={nuevoEstudiante.nombre} />
            <label for="apellido">Apellido</label>
            <input id="apellido" class="form-control mb-2" bind:value={nuevoEstudiante.apellido} />
            <label for="correo">Correo</label>
            <input id="correo" class="form-control mb-2" bind:value={nuevoEstudiante.correo} />
            <label for="fecha_nacimiento">Fecha de Nacimiento</label>
            <input id="fecha_nacimiento" class="form-control mb-2" type="date" bind:value={nuevoEstudiante.fecha_nacimiento} />
            <label for="carrera">Carrera</label>
            <input id="carrera" class="form-control mb-2" bind:value={nuevoEstudiante.carrera} />
          {:else if modalEntidad === 'profesor'}
            <label for="nombre">Nombre</label>
            <input id="nombre" class="form-control mb-2" bind:value={nuevoProfesor.nombre} />
            <label for="apellido">Apellido</label>
            <input id="apellido" class="form-control mb-2" bind:value={nuevoProfesor.apellido} />
            <label for="correo">Correo</label>
            <input id="correo" class="form-control mb-2" bind:value={nuevoProfesor.correo} />
            <label for="especialidad">Especialidad</label>
            <input id="especialidad" class="form-control mb-2" bind:value={nuevoProfesor.especialidad} />
          {:else if modalEntidad === 'curso'}
            <label for="nombre">Nombre</label>
            <input id="nombre" class="form-control mb-2" bind:value={nuevoCurso.nombre} />
            <label for="codigo">Código</label>
            <input id="codigo" class="form-control mb-2" bind:value={nuevoCurso.codigo} />
            <label for="creditos">Créditos</label>
            <input id="creditos" class="form-control mb-2" type="number" bind:value={nuevoCurso.creditos} />
            <select class="form-control mb-2" bind:value={nuevoCurso.id_profesor}>
              <option value="">Seleccione profesor</option>
              {#each profesores as p}
                <option value={p.id_profesor}>{p.nombre} {p.apellido}</option>
              {/each}
            </select>
          {:else if modalEntidad === 'evaluacion'}
            <label for="matricula">Matrícula</label>
            <select id="matricula" class="form-control mb-2" bind:value={nuevaEvaluacion.id_matricula}>
              <option value="">Seleccione matrícula</option>
              {#each matriculas as m}
                <option value={m.id_matricula}>
                  {obtenerEstudianteNombre(m.id_estudiante)} - {obtenerNombreCurso(m.id_curso)}
                </option>
              {/each}
            </select>

            <label for="tipoEvaluacion">Tipo</label>
            <input id="tipoEvaluacion" class="form-control mb-2" placeholder="Tipo" bind:value={nuevaEvaluacion.tipo} />

            <label for="notaEvaluacion">Nota</label>
            <input id="notaEvaluacion" class="form-control mb-2" type="number" placeholder="Nota" bind:value={nuevaEvaluacion.nota} />

            <label for="fechaEvaluacion">Fecha</label>
            <input id="fechaEvaluacion" class="form-control mb-2" type="date" bind:value={nuevaEvaluacion.fecha} />
          {:else if modalEntidad === 'matricula'}
            <select class="form-control mb-2" bind:value={nuevaMatricula.id_estudiante}>
              <option value="">Seleccione estudiante</option>
              {#each estudiantes as e}
                <option value={e.id_estudiante}>{e.nombre} {e.apellido}</option>
              {/each}
            </select>
            <select class="form-control mb-2" bind:value={nuevaMatricula.id_curso}>
              <option value="">Seleccione curso</option>
              {#each cursos as c}
                <option value={c.id_curso}>{c.nombre}</option>
              {/each}
            </select>
            <input class="form-control mb-2" placeholder="Semestre" bind:value={nuevaMatricula.semestre} />
          {/if}
        </div>
        <div class="modal-footer">
          <button class="btn btn-secondary" on:click={() => modalEntidad = ''}>Cancelar</button>
          <button class="btn btn-success" on:click={() => {
            if (modalEntidad === 'matricula') {
              registrarMatricula();
            } else {
              registrarEntidad(modalEntidad);
            }
          }}>Guardar</button>
        </div>
      </div>
    </div>
  </div>
{/if}


</main>




<style>
  .table {
    width: 100%;
    min-width: 100%;
  
  }


  .modal .modal-content {
    max-width: 800px;
    margin: auto;
  }
</style>

