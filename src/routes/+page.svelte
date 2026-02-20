<script lang="ts">
  import "@fontsource/manrope/latin-400.css";
  import "@fontsource/manrope/latin-600.css";
  import "@fontsource/manrope/latin-700.css";
  import "@picocss/pico/css/pico.min.css";
  import "remixicon/fonts/remixicon.css";
  import { onMount, tick } from "svelte";

  type SubjectProfileId = "informatics" | "english";

  type Student = { id: string; name: string };

  type Course = {
    id: string;
    title: string;
    accent: string;
    subjects: { id: string; name: string; profile: SubjectProfileId }[];
    students: Student[];
  };

  type NoteFieldKey = "participacion" | "trabajo" | "carpeta" | "merito";

  type NoteForm = {
    participacion: "" | "activo" | "responde" | "pasivo" | "distraido";
    trabajo: "" | "insuficiente" | "progreso" | "bien" | "muy_bien";
    carpeta: "" | "al_dia" | "incompleta" | "revisar";
    merito: boolean;
    observaciones: string;
  };

  type AssessmentForm = {
    criteria: Record<string, number>;
    criteriaEnabled: Record<string, boolean>;
    trabajoSemana: number;
    trabajoSemanaEnabled: boolean;
    evaluacion: number;
    evaluacionEnabled: boolean;
    comentarios: string;
  };

  type RecordEntry = {
    id: string;
    courseId: string;
    subjectId: string;
    profileId: SubjectProfileId;
    studentId: string;
    studentName: string;
    fields: Record<string, string | number>;
    comment: string;
    timestamp: string;
  };

  const STORAGE_KEY = "registro-estudiantes";
  const COURSES_KEY = "registro-estudiantes-courses";
  const courseAccents = ["#7dd3fc", "#86efac", "#fbbf24", "#f9a8d4", "#c4b5fd", "#6ee7b7"];
  const fieldLabels: Record<string, string> = {
    participacion: "Participacion",
    trabajo: "Trabajo en clase",
    carpeta: "Carpeta",
    merito: "Meritos",
    listening: "Listening",
    speaking: "Speaking",
    reading: "Reading",
    writing: "Writing",
    grammar: "Grammar",
    vocab: "Vocabulario",
    trabajo_semana: "Trabajo semanal",
    evaluacion: "Evaluacion",
    logica: "Logica y problemas",
    practica: "Practica en clase",
    proyecto: "Proyecto semanal",
    documentacion: "Documentacion / carpeta",
  };

  const participationOptions = [
    { id: "distraido", label: "Distracciones", tone: "bad" },
    { id: "pasivo", label: "Pasivo", tone: "mid" },
    { id: "responde", label: "Responde", tone: "ok" },
    { id: "activo", label: "Activo", tone: "good" },
  ] as const;

  const workOptions = [
    { id: "insuficiente", label: "Insuficiente", tone: "bad" },
    { id: "progreso", label: "En progreso", tone: "mid" },
    { id: "bien", label: "Bien", tone: "ok" },
    { id: "muy_bien", label: "Muy bien", tone: "good" },
  ] as const;

  const folderOptions = [
    { id: "revisar", label: "Revisar", tone: "mid" },
    { id: "incompleta", label: "Incompleta", tone: "bad" },
    { id: "al_dia", label: "Al dia", tone: "good" },
  ] as const;

  const englishCriteria = [
    { id: "listening", label: "Listening" },
    { id: "speaking", label: "Speaking" },
    { id: "reading", label: "Reading" },
    { id: "writing", label: "Writing" },
    { id: "grammar", label: "Grammar" },
    { id: "vocab", label: "Vocabulary" },
  ];

  const informaticsCriteria = [
    { id: "logica", label: "Logica y problemas" },
    { id: "practica", label: "Practica en clase" },
    { id: "proyecto", label: "Proyecto semanal" },
    { id: "documentacion", label: "Documentacion / carpeta" },
  ];

  const scaleTen = [
    { value: 1, label: "1" },
    { value: 2, label: "2" },
    { value: 3, label: "3" },
    { value: 4, label: "4" },
    { value: 5, label: "5" },
    { value: 6, label: "6" },
    { value: 7, label: "7" },
    { value: 8, label: "8" },
    { value: 9, label: "9" },
    { value: 10, label: "10" },
  ];

  const assessmentTemplates: Record<
    SubjectProfileId,
    {
      criteria: { id: string; label: string }[];
      scale: { value: number; label: string }[];
    }
  > = {
    english: {
      criteria: englishCriteria,
      scale: scaleTen,
    },
    informatics: {
      criteria: informaticsCriteria,
      scale: scaleTen,
    },
  };

  const defaultNoteForm = (): NoteForm => ({
    participacion: "",
    trabajo: "",
    carpeta: "",
    merito: false,
    observaciones: "",
  });

  const defaultAssessment = (profileId: SubjectProfileId): AssessmentForm => {
    const template = assessmentTemplates[profileId];
    const criteria: Record<string, number> = {};
    const criteriaEnabled: Record<string, boolean> = {};
    template.criteria.forEach((item) => {
      const middle =
        template.scale[Math.floor(template.scale.length / 2)]?.value ?? 1;
      criteria[item.id] = middle;
      criteriaEnabled[item.id] = false;
    });
    return {
      criteria,
      criteriaEnabled,
      trabajoSemana: 5,
      trabajoSemanaEnabled: false,
      evaluacion: 5,
      evaluacionEnabled: false,
      comentarios: "",
    };
  };

  const buildStudents = (prefix: string, names: string[]): Student[] =>
    names.map((name, index) => ({
      id: `${prefix}-${index + 1}`,
      name,
    }));

  const baseCourses: Course[] = [
    {
      id: "es1",
      title: "ES1 Informatica",
      accent: "#7dd3fc",
      subjects: [
        { id: "informatica-es1", name: "Informatica", profile: "informatics" },
      ],
      students: buildStudents("es1", [
        "Ana Alvarez",
        "Bruno Bianchi",
        "Camila Costa",
        "Diego Diaz",
        "Elena Espinoza",
        "Fabricio Flores",
        "Gisela Gimenez",
        "Hernan Herrera",
      ]),
    },
    {
      id: "es2",
      title: "ES2 Informatica",
      accent: "#c084fc",
      subjects: [
        { id: "informatica-es2", name: "Informatica", profile: "informatics" },
      ],
      students: buildStudents("es2", [
        "Ian Ibanez",
        "Juana Jofre",
        "Kevin Kranevitter",
        "Lucia Ledesma",
        "Matias Mendez",
        "Noelia Nieva",
        "Omar Oviedo",
        "Pilar Peralta",
      ]),
    },
    {
      id: "es3",
      title: "ES3 Informatica",
      accent: "#4ade80",
      subjects: [
        { id: "informatica-es3", name: "Informatica", profile: "informatics" },
      ],
      students: buildStudents("es3", [
        "Quimey Quiroga",
        "Rocio Rivas",
        "Santiago Sosa",
        "Tamara Torres",
        "Ulises Ulloa",
        "Valeria Vera",
        "Wanda Wierzba",
        "Zoe Zapata",
      ]),
    },
    {
      id: "ingles",
      title: "Ingles",
      accent: "#fbbf24",
      subjects: [{ id: "ingles", name: "Ingles", profile: "english" }],
      students: buildStudents("en", [
        "Aaron Aguilar",
        "Belen Benitez",
        "Cristian Calvo",
        "Daniela Duarte",
        "Emilia Escalante",
        "Franco Funes",
        "Guadalupe Gallo",
        "Helena Heredia",
      ]),
    },
  ];

  const cloneCourses = () =>
    JSON.parse(JSON.stringify(baseCourses)) as Course[];

  let courses: Course[] = cloneCourses();
  let selectedCourseId = courses[0].id;
  let selectedSubjectId = courses[0].subjects[0].id;
  let activeStudentId = "";
  let filterText = "";
  let filterDebounced = "";
  let filterTimer: ReturnType<typeof setTimeout> | null = null;
  let csvInput: HTMLInputElement;
  let jsonInput: HTMLInputElement;
  let importFeedback = "";
  let editingCourseId: string | null = null;
  let editingCourseName = "";
  let noteForm: NoteForm = defaultNoteForm();
  let assessmentForm: AssessmentForm = defaultAssessment(
    courses[0].subjects[0].profile,
  );
  let records: RecordEntry[] = [];
  let hydrated = false;
  let selectedCourse: Course = courses[0];
  let subjectOptions = selectedCourse.subjects;
  let selectedSubject = subjectOptions[0];
  let filteredStudents = selectedCourse.students;
  let activeRecords: RecordEntry[] = [];
  let showModal = false;
  let modalTab: "nota" | "calificacion" | "historial" = "nota";
  let currentProfileId: SubjectProfileId =
    selectedSubject?.profile ?? "informatics";

  const noteKeys: NoteFieldKey[] = [
    "participacion",
    "trabajo",
    "carpeta",
    "merito",
  ];
  let dateFrom = "";
  let dateTo = "";
  let exportError = "";
  let participationLabel = "";
  let workLabel = "";
  let folderLabel = "";
  let showExportModal = false;
  let exportTempFrom = "";
  let exportTempTo = "";

  // Toast state
  let toastMessage = "";
  let toastVisible = false;
  let toastTimer: ReturnType<typeof setTimeout> | null = null;

  // History lazy load
  let historyLimit = 30;

  // Student management
  let addingStudent = false;
  let newStudentName = "";
  let editingStudentId: string | null = null;
  let editingStudentName = "";

  // Delete confirmation
  let deleteConfirmId: string | null = null;

  function showToast(msg: string) {
    toastMessage = msg;
    toastVisible = true;
    if (toastTimer) clearTimeout(toastTimer);
    toastTimer = setTimeout(() => {
      toastVisible = false;
    }, 2000);
  }

  function handleFilterInput(value: string) {
    filterText = value;
    if (filterTimer) clearTimeout(filterTimer);
    filterTimer = setTimeout(() => {
      filterDebounced = value;
    }, 150);
  }

  onMount(() => {
    hydrated = true;
    const storedCourses = localStorage.getItem(COURSES_KEY);
    if (storedCourses) {
      try {
        const parsed = JSON.parse(storedCourses) as Course[];
        if (parsed.length) courses = parsed;
      } catch {}
    }
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      try {
        records = JSON.parse(stored) as RecordEntry[];
      } catch (error) {
        console.error("No se pudo leer registros guardados", error);
      }
    }
    resetNoteForm();
    setDefaultDateRange();
    filterDebounced = "";
  });

  $: if (hydrated) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(records));
  }

  $: if (hydrated) {
    localStorage.setItem(COURSES_KEY, JSON.stringify(courses));
  }

  $: selectedCourse =
    courses.find((course) => course.id === selectedCourseId) ??
    courses[0] ??
    ({} as Course);
  $: subjectOptions = selectedCourse?.subjects ?? [];
  $: selectedSubject =
    subjectOptions.find((subject) => subject.id === selectedSubjectId) ??
    subjectOptions[0];
  $: currentProfileId = selectedSubject?.profile ?? "informatics";

  $: if (currentProfileId && assessmentTemplates[currentProfileId]) {
    const expected = assessmentTemplates[currentProfileId].criteria.map(
      (item) => item.id,
    );
    const missing = expected.some(
      (id) => assessmentForm.criteria[id] === undefined,
    );
    if (missing) {
      resetAssessment(currentProfileId);
    }
  }

  $: if (
    !subjectOptions.find((subject) => subject.id === selectedSubjectId) &&
    subjectOptions[0]
  ) {
    selectedSubjectId = subjectOptions[0].id;
  }

  $: participationLabel =
    noteValueFor("participacion", noteForm.participacion) ?? "";
  $: workLabel = noteValueFor("trabajo", noteForm.trabajo) ?? "";
  $: folderLabel = noteValueFor("carpeta", noteForm.carpeta) ?? "";

  $: if (selectedCourse && activeStudentId) {
    const exists = selectedCourse.students.some(
      (student) => student.id === activeStudentId,
    );
    if (!exists) {
      activeStudentId = "";
      showModal = false;
    }
  }

  $: filteredStudents = (selectedCourse?.students ?? []).filter((student) =>
    student.name.toLowerCase().includes(filterDebounced.trim().toLowerCase()),
  );

  $: activeRecords = records.filter(
    (entry) =>
      entry.studentId === activeStudentId &&
      entry.courseId === selectedCourseId &&
      entry.subjectId === selectedSubjectId,
  );

  // Count records per student for today
  function studentTodayCount(studentId: string): number {
    const today = new Date().toISOString().slice(0, 10);
    return records.filter(
      (r) =>
        r.studentId === studentId &&
        r.courseId === selectedCourseId &&
        r.timestamp.slice(0, 10) === today,
    ).length;
  }

  function studentTotalCount(studentId: string): number {
    return records.filter(
      (r) =>
        r.studentId === studentId &&
        r.courseId === selectedCourseId,
    ).length;
  }

  function resetNoteForm() {
    noteForm = defaultNoteForm();
  }

  function resetAssessment(profileId: SubjectProfileId) {
    assessmentForm = defaultAssessment(profileId);
  }

  function updateNoteForm(patch: Partial<NoteForm>) {
    noteForm = { ...noteForm, ...patch };
  }

  function setCriterion(fieldId: string, value: number) {
    assessmentForm = {
      ...assessmentForm,
      criteria: { ...assessmentForm.criteria, [fieldId]: value },
      criteriaEnabled: { ...assessmentForm.criteriaEnabled, [fieldId]: true },
    };
  }

  function toggleCriterion(fieldId: string) {
    assessmentForm = {
      ...assessmentForm,
      criteriaEnabled: {
        ...assessmentForm.criteriaEnabled,
        [fieldId]: !assessmentForm.criteriaEnabled[fieldId],
      },
    };
  }

  function toggleAssessmentField(
    field: "trabajoSemanaEnabled" | "evaluacionEnabled",
  ) {
    assessmentForm = {
      ...assessmentForm,
      [field]: !assessmentForm[field],
    } as AssessmentForm;
  }

  function setAssessmentNumber(
    field: "trabajoSemana" | "evaluacion",
    value: number,
  ) {
    const flag =
      field === "trabajoSemana" ? "trabajoSemanaEnabled" : "evaluacionEnabled";
    assessmentForm = {
      ...assessmentForm,
      [field]: value,
      [flag]: true,
    } as AssessmentForm;
  }

  function setAssessmentComment(value: string) {
    assessmentForm = { ...assessmentForm, comentarios: value };
  }

  function setCourse(courseId: string) {
    selectedCourseId = courseId;
    const course = courses.find((item) => item.id === courseId);
    const firstSubject = course?.subjects?.[0];
    if (firstSubject) {
      selectedSubjectId = firstSubject.id;
    }
    activeStudentId = "";
    showModal = false;
    modalTab = "nota";
    resetNoteForm();
    resetAssessment(firstSubject?.profile ?? "informatics");
  }

  async function openStudentModal(studentId: string) {
    activeStudentId = studentId;
    showModal = true;
    modalTab = "nota";
    historyLimit = 30;
    resetNoteForm();
    resetAssessment(currentProfileId);
    await tick();
    const modal = document.querySelector('.modal');
    if (modal) modal.scrollTop = 0;
  }

  function closeModal() {
    showModal = false;
  }

  // Navigate to next/previous student
  function navigateStudent(direction: 1 | -1) {
    const students = selectedCourse.students;
    const currentIndex = students.findIndex((s) => s.id === activeStudentId);
    if (currentIndex === -1) return;
    const nextIndex = (currentIndex + direction + students.length) % students.length;
    openStudentModal(students[nextIndex].id);
  }

  function startCourseEdit(course: Course) {
    editingCourseId = course.id;
    editingCourseName = course.title;
  }

  function saveCourseEdit() {
    const name = editingCourseName.trim();
    if (name && editingCourseId) {
      courses = courses.map((c) =>
        c.id !== editingCourseId ? c : { ...c, title: name },
      );
    }
    editingCourseId = null;
  }

  function deleteCourse(courseId: string) {
    if (courses.length <= 1) return;
    courses = courses.filter((c) => c.id !== courseId);
    if (selectedCourseId === courseId) selectedCourseId = courses[0].id;
  }

  function addCourse() {
    const id = `course-${Date.now().toString(36)}`;
    const accent = courseAccents[courses.length % courseAccents.length];
    const newCourse: Course = {
      id,
      title: "Nuevo curso",
      accent,
      subjects: [{ id: `${id}-subj`, name: "Nuevo curso", profile: "informatics" }],
      students: [],
    };
    courses = [...courses, newCourse];
    editingCourseId = id;
    editingCourseName = "Nuevo curso";
  }

  // Student management
  function addStudent() {
    const name = newStudentName.trim();
    if (!name) return;
    const timestamp = Date.now().toString(36);
    const student: Student = {
      id: `${selectedCourseId}-${timestamp}`,
      name,
    };
    courses = courses.map((c) =>
      c.id !== selectedCourseId ? c : { ...c, students: [...c.students, student] },
    );
    newStudentName = "";
    addingStudent = false;
    showToast(`${name} agregado`);
  }

  function startEditStudent(student: Student) {
    editingStudentId = student.id;
    editingStudentName = student.name;
  }

  function saveStudentEdit() {
    const name = editingStudentName.trim();
    if (name && editingStudentId) {
      courses = courses.map((c) =>
        c.id !== selectedCourseId
          ? c
          : {
              ...c,
              students: c.students.map((s) =>
                s.id !== editingStudentId ? s : { ...s, name },
              ),
            },
      );
    }
    editingStudentId = null;
  }

  function deleteStudent(studentId: string) {
    courses = courses.map((c) =>
      c.id !== selectedCourseId
        ? c
        : { ...c, students: c.students.filter((s) => s.id !== studentId) },
    );
    if (activeStudentId === studentId) {
      activeStudentId = "";
      showModal = false;
    }
    showToast("Alumno eliminado");
  }

  $: if (selectedCourseId) importFeedback = "";

  function handleCSVFile(event: Event) {
    importFeedback = "";
    const courseId = selectedCourseId;
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;
    const currentCount = selectedCourse.students.length;
    const reader = new FileReader();
    reader.onload = (e) => {
      const text = e.target?.result as string;
      const students = parseCSVStudents(text, courseId);
      if (!students.length) {
        importFeedback = "No se encontraron alumnos validos en el archivo.";
      } else {
        if (currentCount > 0) {
          const ok = confirm(
            `Esto reemplazara los ${currentCount} alumnos actuales con ${students.length} nuevos. ¿Continuar?`,
          );
          if (!ok) {
            importFeedback = "Importacion cancelada.";
            return;
          }
        }
        courses = courses.map((course) =>
          course.id !== courseId ? course : { ...course, students },
        );
        activeStudentId = "";
        showModal = false;
        filterText = "";
        filterDebounced = "";
        importFeedback = `${students.length} alumnos importados.`;
        showToast(`${students.length} alumnos importados`);
      }
    };
    reader.onerror = () => {
      importFeedback = "Error al leer el archivo.";
    };
    reader.readAsText(file, "UTF-8");
    input.value = "";
  }

  function parseCSVStudents(text: string, courseId: string): Student[] {
    const sep = text.includes(";") ? ";" : ",";
    const lines = text.split(/\r?\n/);
    const timestamp = Date.now().toString(36);
    const students: Student[] = [];
    for (const line of lines) {
      const trimmed = line.trim();
      if (!trimmed) continue;
      const parts = trimmed
        .split(sep)
        .map((p) => p.trim().replace(/^["']|["']$/g, ""));
      const apellido = parts[0]?.trim() ?? "";
      const nombre = parts[1]?.trim() ?? "";
      if (!students.length && /apellido|nombre|alumno|student/i.test(apellido))
        continue;
      if (!apellido) continue;
      const fullName = nombre ? `${apellido} ${nombre}` : apellido;
      students.push({
        id: `${courseId}-${timestamp}-${students.length}`,
        name: fullName,
      });
    }
    return students;
  }

  function noteValueFor(field: NoteFieldKey, current?: string | boolean) {
    const source = current ?? noteForm[field];
    if (field === "merito") {
      return source ? "Merito" : "";
    }
    if (field === "participacion") {
      return (
        participationOptions.find((option) => option.id === source)?.label ?? ""
      );
    }
    if (field === "trabajo") {
      return workOptions.find((option) => option.id === source)?.label ?? "";
    }
    if (field === "carpeta") {
      return folderOptions.find((option) => option.id === source)?.label ?? "";
    }
    return "";
  }

  function saveQuickNotes() {
    if (!activeStudentId || !selectedSubjectId) return;
    const student = selectedCourse.students.find(
      (item) => item.id === activeStudentId,
    );
    if (!student) return;

    const enabledKeys = noteKeys.filter((key) => {
      if (key === "merito") return noteForm.merito;
      return Boolean(noteForm[key]);
    });
    if (!enabledKeys.length && !noteForm.observaciones.trim()) return;

    const now = new Date();
    const newEntries: RecordEntry[] = [];

    enabledKeys.forEach((key) => {
      const currentValue =
        key === "merito" ? noteForm.merito : (noteForm as any)[key];
      const value = noteValueFor(key, currentValue);
      if (!value) return;
      newEntries.push({
        id: `${now.getTime().toString(36)}-${Math.random().toString(36).slice(2, 7)}-${key}`,
        courseId: selectedCourseId,
        subjectId: selectedSubjectId,
        profileId: selectedSubject?.profile ?? "informatics",
        studentId: activeStudentId,
        studentName: student.name,
        fields: { [key]: value },
        comment: noteForm.observaciones.trim(),
        timestamp: now.toISOString(),
      });
    });

    if (!newEntries.length) return;

    records = [...newEntries, ...records];
    resetNoteForm();
    showToast("Nota guardada");
  }

  function saveAssessment() {
    if (!activeStudentId || !selectedSubjectId) return;
    const student = selectedCourse.students.find(
      (item) => item.id === activeStudentId,
    );
    if (!student) return;
    const template = assessmentTemplates[currentProfileId];
    const now = new Date();
    const fields: Record<string, string | number> = {};

    template.criteria.forEach((item) => {
      if (!assessmentForm.criteriaEnabled[item.id]) return;
      fields[item.id] = assessmentForm.criteria[item.id];
    });

    if (assessmentForm.trabajoSemanaEnabled) {
      fields.trabajo_semana =
        Math.round(assessmentForm.trabajoSemana * 10) / 10;
    }

    if (assessmentForm.evaluacionEnabled) {
      fields.evaluacion = Math.round(assessmentForm.evaluacion * 10) / 10;
    }

    if (!Object.keys(fields).length && !assessmentForm.comentarios.trim()) return;

    const entry: RecordEntry = {
      id: `${now.getTime().toString(36)}-${Math.random().toString(36).slice(2, 7)}`,
      courseId: selectedCourseId,
      subjectId: selectedSubjectId,
      profileId: currentProfileId,
      studentId: activeStudentId,
      studentName: student.name,
      fields,
      comment: assessmentForm.comentarios.trim(),
      timestamp: now.toISOString(),
    };

    records = [entry, ...records];
    resetAssessment(currentProfileId);
    showToast("Calificacion guardada");
  }

  function deleteRecord(id: string) {
    if (deleteConfirmId === id) {
      records = records.filter((entry) => entry.id !== id);
      deleteConfirmId = null;
      showToast("Registro eliminado");
    } else {
      deleteConfirmId = id;
      setTimeout(() => {
        if (deleteConfirmId === id) deleteConfirmId = null;
      }, 3000);
    }
  }

  function setDefaultDateRange() {
    const today = new Date();
    const past = new Date();
    past.setDate(today.getDate() - 30);
    dateFrom = past.toISOString().slice(0, 10);
    dateTo = today.toISOString().slice(0, 10);
  }

  function formatDateInput(value: string) {
    return value.slice(0, 10);
  }

  function openExportModal() {
    setDefaultDateRange();
    exportTempFrom = dateFrom;
    exportTempTo = dateTo;
    exportError = "";
    showExportModal = true;
  }

  function closeExportModal() {
    showExportModal = false;
  }

  function fieldLabel(fieldId: string) {
    return fieldLabels[fieldId] ?? fieldId;
  }

  function getCourseTitleById(courseId: string): string {
    return courses.find((c) => c.id === courseId)?.title ?? courseId.toUpperCase();
  }

  function buildExportData() {
    exportError = "";
    const from = exportTempFrom;
    const to = exportTempTo;
    if (!records.length) {
      exportError = "No hay registros para exportar.";
      return null;
    }
    if (!from || !to) {
      exportError = "Marca rango de fechas.";
      return null;
    }
    const start = new Date(from);
    const end = new Date(to);
    end.setHours(23, 59, 59, 999);
    if (isNaN(start.getTime()) || isNaN(end.getTime()) || start > end) {
      exportError = "Rango de fechas invalido.";
      return null;
    }

    const filtered = records.filter((entry) => {
      const when = new Date(entry.timestamp);
      return when >= start && when <= end;
    });

    if (!filtered.length) {
      exportError = "No hay registros en ese rango.";
      return null;
    }

    const baseFieldOrder = [
      "participacion",
      "trabajo",
      "carpeta",
      "merito",
      "trabajo_semana",
      "evaluacion",
      "logica",
      "practica",
      "proyecto",
      "documentacion",
      "listening",
      "speaking",
      "reading",
      "writing",
      "grammar",
      "vocab",
    ];

    const fieldKeys = new Set<string>(baseFieldOrder);
    filtered.forEach((entry) => {
      Object.keys(entry.fields).forEach((key) => fieldKeys.add(key));
    });
    const sortedFieldKeys = Array.from(fieldKeys);

    type Agg = {
      student: string;
      course: string;
      subject: string;
      fields: Record<string, { sum: number; count: number; lastText: string }>;
    };

    const aggregator = new Map<string, Agg>();

    filtered.forEach((entry) => {
      const key = `${entry.courseId}-${entry.studentId}-${entry.subjectId}`;
      if (!aggregator.has(key)) {
        aggregator.set(key, {
          student: entry.studentName,
          course: getCourseTitleById(entry.courseId),
          subject: entry.profileId === "informatics" ? "Informatica" : "Ingles",
          fields: {},
        });
      }
      const agg = aggregator.get(key)!;
      Object.entries(entry.fields).forEach(([fieldId, raw]) => {
        const current = agg.fields[fieldId] ?? {
          sum: 0,
          count: 0,
          lastText: "",
        };
        const numericValue =
          typeof raw === "number"
            ? raw
            : Number.isFinite(Number(raw))
              ? Number(raw)
              : null;
        if (numericValue !== null) {
          current.sum += numericValue;
          current.count += 1;
        } else if (typeof raw === "string") {
          current.lastText = raw.replace(/[^\x20-\x7E\u00A1-\u00FF]/g, "").trim();
        }
        agg.fields[fieldId] = current;
      });
    });

    const headers = [
      "Curso",
      "Materia",
      "Estudiante",
      ...sortedFieldKeys.map((key) => fieldLabel(key)),
    ];
    const rows: string[][] = [headers];

    aggregator.forEach((agg) => {
      const row: string[] = [agg.course, agg.subject, agg.student];
      sortedFieldKeys.forEach((fieldId) => {
        const data = agg.fields[fieldId];
        if (!data) {
          row.push("");
          return;
        }
        if (data.count > 0) {
          const avg = data.sum / data.count;
          row.push(String(Math.round(avg * 100) / 100));
        } else {
          row.push(data.lastText);
        }
      });
      rows.push(row);
    });

    return { rows, from, to };
  }

  function exportWorkbook() {
    const data = buildExportData();
    if (!data) return;
    const { rows, from, to } = data;

    const tableHtml = rows
      .map(
        (cols) =>
          `<tr>${cols
            .map(
              (col) =>
                `<td style="border:1px solid #ccc;padding:6px;">${col ?? ""}</td>`,
            )
            .join("")}</tr>`,
      )
      .join("");
    const html = `<html><head><meta charset="UTF-8"></head><body><table>${tableHtml}</table></body></html>`;
    const blob = new Blob([html], { type: "application/vnd.ms-excel" });
    downloadBlob(blob, `registro-estudiantes_${from}_a_${to}.xls`);
    showToast("Archivo XLS exportado");
    closeExportModal();
  }

  function exportCSV() {
    const data = buildExportData();
    if (!data) return;
    const { rows, from, to } = data;

    const csvContent = rows
      .map((row) =>
        row.map((cell) => `"${(cell ?? "").replace(/"/g, '""')}"`).join(","),
      )
      .join("\n");
    const bom = "\uFEFF";
    const blob = new Blob([bom + csvContent], { type: "text/csv;charset=utf-8" });
    downloadBlob(blob, `registro-estudiantes_${from}_a_${to}.csv`);
    showToast("Archivo CSV exportado");
    closeExportModal();
  }

  function downloadBlob(blob: Blob, filename: string) {
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = filename;
    link.click();
    URL.revokeObjectURL(url);
  }

  // Backup & Restore
  function exportBackup() {
    const data = {
      version: 1,
      timestamp: new Date().toISOString(),
      courses,
      records,
    };
    const blob = new Blob([JSON.stringify(data, null, 2)], {
      type: "application/json",
    });
    downloadBlob(blob, `backup-registro_${new Date().toISOString().slice(0, 10)}.json`);
    showToast("Backup exportado");
  }

  function handleJsonImport(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const data = JSON.parse(e.target?.result as string);
        if (data.courses && Array.isArray(data.courses)) {
          const ok = confirm(
            `Se importaran ${data.courses.length} cursos y ${data.records?.length ?? 0} registros. Esto reemplazara todos los datos actuales. ¿Continuar?`,
          );
          if (!ok) return;
          courses = data.courses;
          if (data.records) records = data.records;
          selectedCourseId = courses[0]?.id ?? "";
          selectedSubjectId = courses[0]?.subjects?.[0]?.id ?? "";
          activeStudentId = "";
          showModal = false;
          showToast("Backup restaurado");
        } else {
          showToast("Archivo de backup invalido");
        }
      } catch {
        showToast("Error al leer backup");
      }
    };
    reader.readAsText(file, "UTF-8");
    input.value = "";
  }

  const formatStamp = (iso: string) => {
    const date = new Date(iso);
    return `${date.toLocaleDateString()} - ${date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })}`;
  };

  const initials = (name: string) =>
    name
      .split(" ")
      .map((chunk) => chunk[0])
      .join("")
      .slice(0, 2)
      .toUpperCase();
</script>

<svelte:head>
  <title>Registro de estudiantes</title>
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
</svelte:head>

<!-- Toast -->
{#if toastVisible}
  <div class="toast" class:show={toastVisible}>
    <i class="ri-check-line" aria-hidden="true"></i>
    {toastMessage}
  </div>
{/if}

<main class="page">
  <header class="hero">
    <div>
      <p class="eyebrow">Registro en aula</p>
      <h1>Control rapido de alumnos</h1>
      <p class="lede">
        Elegi un curso, toca un estudiante y usa la ficha rapida para dejar
        meritos, observaciones y exportar todo con fecha y hora.
      </p>
      <div class="hero-actions">
        <button
          class="primary"
          type="button"
          on:click={openExportModal}
          disabled={!records.length}
        >
          <i class="ri-file-text-line" aria-hidden="true"></i>
          <span>Exportar</span>
        </button>
        <button class="ghost" type="button" on:click={exportBackup}>
          <i class="ri-download-2-line" aria-hidden="true"></i>
          <span>Backup</span>
        </button>
        <input
          type="file"
          accept=".json"
          bind:this={jsonInput}
          on:change={handleJsonImport}
          style="display:none"
        />
        <button class="ghost" type="button" on:click={() => jsonInput.click()}>
          <i class="ri-upload-2-line" aria-hidden="true"></i>
          <span>Restaurar</span>
        </button>
      </div>
    </div>
  </header>

  {#if showExportModal}
    <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
    <div
      class="modal-backdrop export"
      on:click={(event) => {
        if (event.target === event.currentTarget) closeExportModal();
      }}
    >
      <!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
      <div class="export-card" on:click|stopPropagation>
        <div class="export-header">
          <div>
            <p class="eyebrow">Exportar hoja</p>
            <h4>Selecciona rango</h4>
          </div>
          <button
            class="ghost icon"
            type="button"
            on:click={closeExportModal}
            aria-label="Cerrar exportar"
          >
            <i class="ri-close-line" aria-hidden="true"></i>
          </button>
        </div>
        <div class="export-grid">
          <label>
            <span>Desde</span>
            <input
              type="date"
              bind:value={exportTempFrom}
              on:change={(e) =>
                (exportTempFrom = formatDateInput(e.currentTarget.value))}
            />
          </label>
          <label>
            <span>Hasta</span>
            <input
              type="date"
              bind:value={exportTempTo}
              on:change={(e) =>
                (exportTempTo = formatDateInput(e.currentTarget.value))}
            />
          </label>
        </div>
        {#if exportError}
          <p class="error-text">{exportError}</p>
        {/if}
        <div class="actions export-actions">
          <button class="ghost" type="button" on:click={closeExportModal}
            >Cancelar</button
          >
          <button class="primary" type="button" on:click={exportCSV}>
            <i class="ri-file-list-3-line" aria-hidden="true"></i>
            <span>CSV</span>
          </button>
          <button class="primary" type="button" on:click={exportWorkbook}>
            <i class="ri-file-excel-2-line" aria-hidden="true"></i>
            <span>Excel</span>
          </button>
        </div>
      </div>
    </div>
  {/if}

  <section class="panel">
    <div class="panel-header">
      <div>
        <p class="eyebrow">Paso 1</p>
        <h2>Elegir curso</h2>
      </div>
    </div>

    <div class="course-grid">
      {#each courses as course}
        <div class="course-wrap">
          {#if editingCourseId === course.id}
            <div class="course-editing" style={`--accent:${course.accent}`}>
              <input
                class="course-name-input"
                bind:value={editingCourseName}
                on:blur={saveCourseEdit}
                on:keydown={(e) => {
                  if (e.key === "Enter") saveCourseEdit();
                  if (e.key === "Escape") editingCourseId = null;
                }}
              />
              {#if courses.length > 1}
                <button
                  type="button"
                  class="course-delete-btn"
                  on:mousedown|preventDefault
                  on:click={() => deleteCourse(course.id)}
                  title="Eliminar curso"
                >
                  <i class="ri-delete-bin-line"></i>
                </button>
              {/if}
            </div>
          {:else}
            <button
              type="button"
              class:active={course.id === selectedCourseId}
              style={`--accent:${course.accent}`}
              on:click={() => setCourse(course.id)}
            >
              <span class="course-pill">{course.title}</span>
              <span class="course-count">{course.students.length} alumnos</span>
            </button>
            <button
              type="button"
              class="course-edit-btn"
              on:click|stopPropagation={() => startCourseEdit(course)}
              title="Renombrar"
            >
              <i class="ri-pencil-line"></i>
            </button>
          {/if}
        </div>
      {/each}
      <button type="button" class="add-course-btn" on:click={addCourse} aria-label="Agregar curso">
        <i class="ri-add-line"></i>
      </button>
    </div>
  </section>

  <section class="panel">
    <div class="panel-header">
      <div>
        <p class="eyebrow">Paso 2</p>
        <h2>Elegir estudiante</h2>
      </div>
    </div>

    <div class="toolbar">
      <label class="search">
        <i class="ri-search-line search-icon" aria-hidden="true"></i>
        <input
          placeholder="Buscar estudiante..."
          autocomplete="off"
          value={filterText}
          on:input={(e) => handleFilterInput(e.currentTarget.value)}
          name="search"
        />
        {#if filterText}
          <button
            class="search-clear"
            type="button"
            on:click={() => { filterText = ""; filterDebounced = ""; }}
            aria-label="Limpiar busqueda"
          >
            <i class="ri-close-line"></i>
          </button>
        {/if}
      </label>
      <div class="toolbar-btns">
        <input
          type="file"
          accept=".csv"
          bind:this={csvInput}
          on:change={handleCSVFile}
          style="display:none"
        />
        <button class="ghost compact" type="button" on:click={() => csvInput.click()}>
          <i class="ri-upload-2-line"></i> CSV
        </button>
        <button class="ghost compact" type="button" on:click={() => { addingStudent = true; newStudentName = ""; }}>
          <i class="ri-user-add-line"></i> Agregar
        </button>
      </div>
    </div>

    {#if addingStudent}
      <div class="add-student-row">
        <input
          class="add-student-input"
          placeholder="Nombre y apellido..."
          bind:value={newStudentName}
          on:keydown={(e) => {
            if (e.key === "Enter") addStudent();
            if (e.key === "Escape") addingStudent = false;
          }}
        />
        <button class="primary compact" type="button" on:click={addStudent} aria-label="Confirmar agregar alumno">
          <i class="ri-check-line"></i>
        </button>
        <button class="ghost compact" type="button" on:click={() => addingStudent = false} aria-label="Cancelar agregar alumno">
          <i class="ri-close-line"></i>
        </button>
      </div>
    {/if}

    {#if importFeedback}
      <p class="import-feedback">{importFeedback}</p>
    {/if}

    <div class="student-grid">
      {#if filteredStudents.length}
        {#each filteredStudents as student}
          <div class="student-card-wrap">
            {#if editingStudentId === student.id}
              <div class="student-editing">
                <input
                  class="student-name-input"
                  bind:value={editingStudentName}
                  on:blur={saveStudentEdit}
                  on:keydown={(e) => {
                    if (e.key === "Enter") saveStudentEdit();
                    if (e.key === "Escape") editingStudentId = null;
                  }}
                />
                <button
                  type="button"
                  class="student-delete-btn"
                  on:mousedown|preventDefault
                  on:click={() => deleteStudent(student.id)}
                  title="Eliminar alumno"
                >
                  <i class="ri-delete-bin-line"></i>
                </button>
              </div>
            {:else}
              <button
                type="button"
                class="student-card"
                class:active={student.id === activeStudentId}
                on:click={() => openStudentModal(student.id)}
              >
                <span class="avatar" style={`--accent:${selectedCourse.accent}`}>{initials(student.name)}</span>
                <div class="student-info">
                  <p class="name">{student.name}</p>
                  <div class="student-meta">
                    {#if studentTodayCount(student.id) > 0}
                      <span class="today-badge">
                        <span class="today-dot"></span>
                        hoy: {studentTodayCount(student.id)}
                      </span>
                    {/if}
                    {#if studentTotalCount(student.id) > 0}
                      <span class="total-count">{studentTotalCount(student.id)} reg.</span>
                    {/if}
                  </div>
                </div>
                <i class="ri-arrow-right-s-line" aria-hidden="true"></i>
              </button>
              <button
                type="button"
                class="student-edit-btn"
                on:click|stopPropagation={() => startEditStudent(student)}
                title="Editar alumno"
              >
                <i class="ri-pencil-line"></i>
              </button>
            {/if}
          </div>
        {/each}
      {:else}
        <p class="empty">No hay estudiantes en esta vista.</p>
      {/if}
    </div>
  </section>

  {#if showModal}
    <div
      class="modal-backdrop"
      role="presentation"
      tabindex="-1"
      on:click={(event) => {
        if (event.target === event.currentTarget) {
          closeModal();
        }
      }}
      on:keydown={(event) => {
        if (event.key === "Escape") {
          event.preventDefault();
          closeModal();
        }
      }}
    >
      <div
        class="modal"
        role="dialog"
        aria-modal="true"
        aria-label="Ficha rapida del estudiante"
        tabindex="-1"
        on:keydown={(event) => {
          if (event.key === "Escape") {
            event.preventDefault();
            closeModal();
          }
        }}
      >
        <div class="modal-header">
          <div class="modal-header-left">
            <button
              class="nav-btn"
              type="button"
              on:click={() => navigateStudent(-1)}
              aria-label="Alumno anterior"
              title="Anterior"
            >
              <i class="ri-arrow-left-s-line"></i>
            </button>
            <div>
              <p class="eyebrow">Ficha rapida</p>
              <h3>
                {selectedCourse.students.find((s) => s.id === activeStudentId)
                  ?.name}
              </h3>
              <p class="mini muted">{selectedCourse.title}</p>
            </div>
            <button
              class="nav-btn"
              type="button"
              on:click={() => navigateStudent(1)}
              aria-label="Alumno siguiente"
              title="Siguiente"
            >
              <i class="ri-arrow-right-s-line"></i>
            </button>
          </div>
          <button
            class="ghost icon"
            type="button"
            on:click={closeModal}
            aria-label="Cerrar ficha"
          >
            <i class="ri-close-line" aria-hidden="true"></i>
          </button>
        </div>

        <div class="tabs-row">
          <div class="tabs">
            <button
              type="button"
              class:active={modalTab === "nota"}
              on:click={() => (modalTab = "nota")}
            >
              <i class="ri-sticky-note-line tab-icon" aria-hidden="true"></i>
              Notas
            </button>
            <button
              type="button"
              class:active={modalTab === "calificacion"}
              on:click={() => (modalTab = "calificacion")}
            >
              <i class="ri-bar-chart-line tab-icon" aria-hidden="true"></i>
              Calif.
            </button>
            <button
              type="button"
              class:active={modalTab === "historial"}
              on:click={() => { modalTab = "historial"; historyLimit = 30; }}
            >
              <i class="ri-time-line tab-icon" aria-hidden="true"></i>
              Historial
              {#if activeRecords.length > 0}
                <span class="tab-badge">{activeRecords.length}</span>
              {/if}
            </button>
          </div>
        </div>

        {#if modalTab === "nota"}
          <div class="form-grid modal-grid">
            <div class="field">
              <div class="field-head">
                <p class="label">Participacion</p>
                <span class="value highlight">
                  {participationLabel || "Sin marcar"}
                </span>
              </div>
              <div class="pill-group">
                {#each participationOptions as option}
                  <button
                    type="button"
                    class={`tone-${option.tone}`}
                    class:selected={noteForm.participacion === option.id}
                    on:click={() =>
                      updateNoteForm({
                        participacion:
                          noteForm.participacion === option.id ? "" : option.id,
                      })}
                  >
                    <span class={`dot ${option.tone}`}></span>
                    {option.label}
                  </button>
                {/each}
              </div>
            </div>

            <div class="field">
              <div class="field-head">
                <p class="label">Trabajo en clase</p>
                <span class="value highlight">{workLabel || "Sin marcar"}</span>
              </div>
              <div class="pill-group">
                {#each workOptions as option}
                  <button
                    type="button"
                    class={`tone-${option.tone}`}
                    class:selected={noteForm.trabajo === option.id}
                    on:click={() =>
                      updateNoteForm({
                        trabajo:
                          noteForm.trabajo === option.id ? "" : option.id,
                      })}
                  >
                    <span class={`dot ${option.tone}`}></span>
                    {option.label}
                  </button>
                {/each}
              </div>
            </div>

            <div class="field">
              <div class="field-head">
                <p class="label">Carpeta</p>
                <span class="value highlight"
                  >{folderLabel || "Sin marcar"}</span
                >
              </div>
              <div class="pill-group">
                {#each folderOptions as option}
                  <button
                    type="button"
                    class={`tone-${option.tone}`}
                    class:selected={noteForm.carpeta === option.id}
                    on:click={() =>
                      updateNoteForm({
                        carpeta:
                          noteForm.carpeta === option.id ? "" : option.id,
                      })}
                  >
                    <span class={`dot ${option.tone}`}></span>
                    {option.label}
                  </button>
                {/each}
              </div>
            </div>

            <div class="field merit-field">
              <div class="field-head">
                <p class="label">Meritos</p>
                <span class="value"
                  >{noteForm.merito ? "Merito" : "Sin merito"}</span
                >
              </div>
              <button
                type="button"
                class={`toggle ${noteForm.merito ? "on" : ""}`}
                on:click={() => updateNoteForm({ merito: !noteForm.merito })}
              >
                {noteForm.merito ? "Quitar merito" : "Dar merito"}
              </button>
            </div>

            <label class="field field-span">
              <span>Observaciones</span>
              <textarea
                rows="2"
                placeholder="Comentario breve"
                bind:value={noteForm.observaciones}
                name="comment"
              ></textarea>
            </label>
          </div>

          <div class="sticky-save">
            <button class="primary save-btn" type="button" on:click={saveQuickNotes}>
              <i class="ri-check-line" aria-hidden="true"></i>
              Guardar nota
            </button>
            <button class="ghost compact" type="button" on:click={resetNoteForm}>
              Limpiar
            </button>
          </div>
        {:else if modalTab === "calificacion"}
          <div class="form-grid modal-grid">
            {#each assessmentTemplates[currentProfileId].criteria as item}
              <div class="field">
                <div class="field-head">
                  <p class="label">{item.label}</p>
                  <span class="value score"
                    >{assessmentForm.criteria[item.id]}</span
                  >
                </div>
                <div class="field-subhead">
                  <label
                    class={`switch ${assessmentForm.criteriaEnabled[item.id] ? "on" : "off"}`}
                  >
                    <input
                      type="checkbox"
                      checked={assessmentForm.criteriaEnabled[item.id]}
                      on:change={() => toggleCriterion(item.id)}
                    />
                    <span>
                      {assessmentForm.criteriaEnabled[item.id]
                        ? "Habilitado"
                        : "Deshabilitado"}
                    </span>
                  </label>
                </div>
                <div class="range-row">
                  <button
                    class="range-btn"
                    type="button"
                    on:click={() => setCriterion(item.id, Math.max(1, assessmentForm.criteria[item.id] - 1))}
                  >-</button>
                  <input
                    type="range"
                    min={assessmentTemplates[currentProfileId].scale[0].value}
                    max={assessmentTemplates[currentProfileId].scale.at(-1)
                      ?.value}
                    step="1"
                    value={assessmentForm.criteria[item.id]}
                    on:input={(event) =>
                      setCriterion(item.id, Number(event.currentTarget.value))}
                  />
                  <button
                    class="range-btn"
                    type="button"
                    on:click={() => setCriterion(item.id, Math.min(10, assessmentForm.criteria[item.id] + 1))}
                  >+</button>
                </div>
              </div>
            {/each}

            <div class="field">
              <div class="field-head">
                <p class="label">Trabajo semanal</p>
                <span class="value score">
                  {Math.round(assessmentForm.trabajoSemana * 10) / 10}
                </span>
              </div>
              <div class="field-subhead">
                <label
                  class={`switch ${assessmentForm.trabajoSemanaEnabled ? "on" : "off"}`}
                >
                  <input
                    type="checkbox"
                    checked={assessmentForm.trabajoSemanaEnabled}
                    on:change={() =>
                      toggleAssessmentField("trabajoSemanaEnabled")}
                  />
                  <span>
                    {assessmentForm.trabajoSemanaEnabled
                      ? "Habilitado"
                      : "Deshabilitado"}
                  </span>
                </label>
              </div>
              <div class="range-row">
                <button
                  class="range-btn"
                  type="button"
                  on:click={() => setAssessmentNumber("trabajoSemana", Math.max(1, assessmentForm.trabajoSemana - 0.5))}
                >-</button>
                <input
                  type="range"
                  min="1"
                  max="10"
                  step="0.5"
                  value={assessmentForm.trabajoSemana}
                  on:input={(event) =>
                    setAssessmentNumber(
                      "trabajoSemana",
                      Number(event.currentTarget.value),
                    )}
                />
                <button
                  class="range-btn"
                  type="button"
                  on:click={() => setAssessmentNumber("trabajoSemana", Math.min(10, assessmentForm.trabajoSemana + 0.5))}
                >+</button>
              </div>
            </div>

            <div class="field">
              <div class="field-head">
                <p class="label">Evaluacion</p>
                <span class="value score">
                  {Math.round(assessmentForm.evaluacion * 10) / 10}
                </span>
              </div>
              <div class="field-subhead">
                <label
                  class={`switch ${assessmentForm.evaluacionEnabled ? "on" : "off"}`}
                >
                  <input
                    type="checkbox"
                    checked={assessmentForm.evaluacionEnabled}
                    on:change={() => toggleAssessmentField("evaluacionEnabled")}
                  />
                  <span>
                    {assessmentForm.evaluacionEnabled
                      ? "Habilitado"
                      : "Deshabilitado"}
                  </span>
                </label>
              </div>
              <div class="range-row">
                <button
                  class="range-btn"
                  type="button"
                  on:click={() => setAssessmentNumber("evaluacion", Math.max(1, assessmentForm.evaluacion - 0.5))}
                >-</button>
                <input
                  type="range"
                  min="1"
                  max="10"
                  step="0.5"
                  value={assessmentForm.evaluacion}
                  on:input={(event) =>
                    setAssessmentNumber(
                      "evaluacion",
                      Number(event.currentTarget.value),
                    )}
                />
                <button
                  class="range-btn"
                  type="button"
                  on:click={() => setAssessmentNumber("evaluacion", Math.min(10, assessmentForm.evaluacion + 0.5))}
                >+</button>
              </div>
            </div>

            <label class="field field-span">
              <span>Notas generales</span>
              <textarea
                rows="2"
                placeholder="Puntos fuertes, pendientes, objetivos de la semana"
                value={assessmentForm.comentarios}
                name="assessment-comment"
                on:input={(event) =>
                  setAssessmentComment(event.currentTarget.value)}
              ></textarea>
            </label>
          </div>

          <div class="sticky-save">
            <button class="primary save-btn" type="button" on:click={saveAssessment}>
              <i class="ri-check-line" aria-hidden="true"></i>
              Guardar calificacion
            </button>
            <button
              class="ghost compact"
              type="button"
              on:click={() => resetAssessment(currentProfileId)}
            >
              Limpiar
            </button>
          </div>
        {:else}
          <div class="modal-history">
            {#if activeRecords.length}
              {#each activeRecords.slice(0, historyLimit) as entry}
                {#each Object.entries(entry.fields) as [key, value]}
                  <div class="history-card">
                    <div class="history-top">
                      <div class="note-line compact">
                        <span class="badge strong">{fieldLabel(key)}</span>
                        <span class="note-value">{value}</span>
                      </div>
                      <button
                        class={`ghost ${deleteConfirmId === entry.id ? "danger-confirm" : "danger"}`}
                        type="button"
                        on:click={() => deleteRecord(entry.id)}
                      >
                        {deleteConfirmId === entry.id ? "Confirmar" : "Borrar"}
                      </button>
                    </div>
                    <span class="stamp small"
                      >{formatStamp(entry.timestamp)}</span
                    >
                    {#if entry.comment}
                      <p class="comment">{entry.comment}</p>
                    {/if}
                  </div>
                {/each}
              {/each}
              {#if activeRecords.length > historyLimit}
                <button
                  class="ghost load-more"
                  type="button"
                  on:click={() => historyLimit += 30}
                >
                  Cargar mas ({activeRecords.length - historyLimit} restantes)
                </button>
              {/if}
            {:else}
              <p class="empty">
                Todavia no hay registros para este estudiante.
              </p>
            {/if}
          </div>
        {/if}
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    font-family:
      "Manrope",
      "Inter",
      system-ui,
      -apple-system,
      BlinkMacSystemFont,
      "Segoe UI",
      sans-serif;
    background: radial-gradient(circle at 10% 10%, #122542, #0b1021 32%),
      #0b1021;
    color: #e8ecf5;
    min-height: 100vh;
    -webkit-tap-highlight-color: transparent;
    -webkit-touch-callout: none;
  }

  :global(select),
  :global(input),
  :global(textarea),
  :global(button) {
    font-family: inherit;
  }

  :global(option) {
    color: #0b1021;
  }

  /* Toast */
  .toast {
    position: fixed;
    top: 1rem;
    left: 50%;
    transform: translateX(-50%) translateY(-20px);
    background: linear-gradient(135deg, #4ade80, #22d3ee);
    color: #0b1224;
    font-weight: 800;
    padding: 0.65rem 1.2rem;
    border-radius: 0.85rem;
    z-index: 100;
    display: flex;
    align-items: center;
    gap: 0.4rem;
    box-shadow: 0 8px 32px rgba(74, 222, 128, 0.3);
    opacity: 0;
    transition: opacity 0.3s, transform 0.3s;
    pointer-events: none;
    font-size: 0.9rem;
  }

  .toast.show {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }

  main.page {
    max-width: 1100px;
    margin: 0 auto;
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .hero {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
    align-items: flex-start;
  }

  h1 {
    margin: 0 0 0.25rem 0;
    font-size: clamp(1.5rem, 3vw, 2.3rem);
    color: #f8fafc;
  }

  h2 {
    margin: 0;
    color: #f3f4f6;
  }

  .lede {
    color: #b7c5e0;
    margin-bottom: 0.75rem;
    font-size: 0.9rem;
  }

  .eyebrow {
    text-transform: uppercase;
    font-size: 0.72rem;
    letter-spacing: 0.08em;
    color: #7dd3fc;
    margin: 0;
  }

  .hero-actions {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
    align-items: flex-end;
  }

  .error-text {
    color: #fecdd3;
    margin: 0.35rem 0 0 0;
    font-weight: 700;
  }

  .label {
    font-size: 0.85rem;
    color: #9fb1cc;
    margin: 0 0 0.15rem 0;
  }

  .value {
    margin: 0;
    color: #f8fafc;
    font-weight: 700;
  }

  .panel {
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 1rem;
    padding: 1rem;
    background: rgba(10, 18, 36, 0.85);
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.35);
    backdrop-filter: blur(6px);
  }

  .panel-header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    gap: 0.75rem;
    margin-bottom: 0.5rem;
  }

  .course-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 0.5rem;
    margin-top: 0.5rem;
  }

  .course-wrap {
    position: relative;
  }

  .course-wrap > button:not(.course-edit-btn) {
    width: 100%;
    border-radius: 0.55rem;
    padding: 0.6rem 0.85rem;
    border: 1px solid rgba(255, 255, 255, 0.08);
    background: rgba(255, 255, 255, 0.03);
    color: #e5e7eb;
    text-align: left;
    font-size: 0.9rem;
    min-height: 44px;
    transition: border-color 0.2s, background 0.2s;
  }

  .course-wrap > button.active {
    border-color: var(--accent, #7dd3fc);
    background: rgba(125, 211, 252, 0.08);
    color: #f8fafc;
  }

  .course-count {
    display: block;
    font-size: 0.72rem;
    color: #8da0bf;
    margin-top: 0.1rem;
  }

  .course-edit-btn {
    position: absolute;
    top: 0.3rem;
    right: 0.3rem;
    padding: 0.2rem 0.3rem;
    border-radius: 0.4rem;
    border: none;
    background: rgba(0, 0, 0, 0.4);
    color: rgba(255, 255, 255, 0.45);
    font-size: 0.75rem;
    line-height: 1;
    min-height: 28px;
    min-width: 28px;
    display: grid;
    place-items: center;
  }

  .course-editing {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    border: 1px solid var(--accent, #7dd3fc);
    border-radius: 0.55rem;
    padding: 0.5rem 0.75rem;
    background: rgba(255, 255, 255, 0.03);
  }

  .course-name-input {
    border: none;
    background: transparent;
    color: #f8fafc;
    font-size: 0.9rem;
    padding: 0;
    width: 100%;
  }

  .course-delete-btn {
    border: none;
    background: transparent;
    color: rgba(248, 113, 113, 0.7);
    padding: 0.1rem 0.2rem;
    font-size: 0.9rem;
    flex-shrink: 0;
    min-height: 44px;
    min-width: 44px;
    display: grid;
    place-items: center;
  }

  .add-course-btn {
    border-radius: 0.55rem;
    padding: 0.75rem;
    border: 1px dashed rgba(255, 255, 255, 0.15);
    background: transparent;
    color: rgba(255, 255, 255, 0.35);
    font-size: 1.1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 44px;
  }

  .course-pill {
    display: block;
    font-weight: 500;
  }

  label {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
    font-weight: 600;
    color: #f3f4f6;
  }

  input,
  textarea {
    background: rgba(255, 255, 255, 0.04);
    color: #e5e7eb;
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 0.65rem;
    padding: 0.65rem;
    width: 100%;
  }

  .toolbar {
    display: flex;
    gap: 0.5rem;
    margin: 0.5rem 0 0.75rem 0;
    align-items: center;
    flex-wrap: wrap;
  }

  .toolbar-btns {
    display: flex;
    gap: 0.35rem;
    flex-shrink: 0;
  }

  .search {
    display: flex;
    align-items: center;
    position: relative;
    flex: 1;
    min-width: 160px;
  }

  .search-icon {
    position: absolute;
    left: 0.65rem;
    color: #8da0bf;
    font-size: 0.9rem;
    pointer-events: none;
  }

  .search input {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(255, 255, 255, 0.12);
    border-radius: 0.75rem;
    padding: 0.5rem 2rem 0.5rem 2rem;
    font-size: 0.85rem;
    color: #f8fafc;
    width: 100%;
    min-height: 44px;
  }

  .search-clear {
    position: absolute;
    right: 0.3rem;
    background: transparent;
    border: none;
    color: #8da0bf;
    padding: 0.3rem;
    border-radius: 50%;
    display: grid;
    place-items: center;
    min-height: 32px;
    min-width: 32px;
  }

  .add-student-row {
    display: flex;
    gap: 0.4rem;
    margin-bottom: 0.75rem;
    align-items: center;
  }

  .add-student-input {
    flex: 1;
    min-height: 44px;
  }

  .import-feedback {
    font-size: 0.8rem;
    margin: -0.25rem 0 0.5rem 0;
    opacity: 0.85;
  }

  .student-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 0.5rem;
  }

  .student-card-wrap {
    position: relative;
  }

  .student-card {
    display: grid;
    grid-template-columns: auto 1fr auto;
    align-items: center;
    gap: 0.5rem;
    padding: 0.6rem 0.75rem;
    border-radius: 0.9rem;
    border: 1px solid rgba(255, 255, 255, 0.08);
    background: rgba(255, 255, 255, 0.03);
    color: #e5e7eb;
    width: 100%;
    min-height: 56px;
    transition: border-color 0.2s, background 0.2s;
  }

  .student-card.active {
    border-color: rgba(74, 222, 128, 0.6);
    background: rgba(74, 222, 128, 0.08);
  }

  .student-info {
    min-width: 0;
  }

  .student-meta {
    display: flex;
    gap: 0.5rem;
    align-items: center;
    flex-wrap: wrap;
  }

  .today-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.25rem;
    font-size: 0.72rem;
    color: #4ade80;
    font-weight: 700;
  }

  .today-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #4ade80;
    display: inline-block;
  }

  .total-count {
    font-size: 0.72rem;
    color: #8da0bf;
  }

  .student-edit-btn {
    position: absolute;
    top: 0.25rem;
    right: 0.25rem;
    padding: 0.15rem 0.25rem;
    border-radius: 0.35rem;
    border: none;
    background: rgba(0, 0, 0, 0.35);
    color: rgba(255, 255, 255, 0.35);
    font-size: 0.7rem;
    line-height: 1;
    min-height: 24px;
    min-width: 24px;
    display: grid;
    place-items: center;
    opacity: 0;
    transition: opacity 0.2s;
  }

  .student-card-wrap:hover .student-edit-btn,
  .student-card-wrap:focus-within .student-edit-btn {
    opacity: 1;
  }

  .student-editing {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    border: 1px solid #7dd3fc;
    border-radius: 0.9rem;
    padding: 0.5rem 0.75rem;
    background: rgba(255, 255, 255, 0.03);
    min-height: 56px;
  }

  .student-name-input {
    border: none;
    background: transparent;
    color: #f8fafc;
    font-size: 0.9rem;
    padding: 0;
    width: 100%;
  }

  .student-delete-btn {
    border: none;
    background: transparent;
    color: rgba(248, 113, 113, 0.7);
    padding: 0.1rem 0.2rem;
    font-size: 0.9rem;
    flex-shrink: 0;
    min-height: 44px;
    min-width: 44px;
    display: grid;
    place-items: center;
  }

  .avatar {
    width: 38px;
    height: 38px;
    border-radius: 10px;
    background: linear-gradient(135deg, var(--accent, #7dd3fc), #4ade80);
    display: grid;
    place-items: center;
    font-weight: 800;
    color: #0b1224;
    font-size: 0.8rem;
    flex-shrink: 0;
  }

  .name {
    margin: 0;
    font-weight: 700;
    color: #f8fafc;
    font-size: 0.9rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .mini {
    margin: 0;
    color: #9fb1cc;
    font-size: 0.85rem;
  }

  .empty {
    margin: 0.35rem 0;
    color: #9fb1cc;
  }

  .form-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 0.6rem;
  }

  .field {
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 0.9rem;
    padding: 0.75rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .field-head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.4rem;
  }

  .field-subhead {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.35rem;
  }

  .value {
    font-weight: 700;
    color: #f8fafc;
  }

  .value.highlight {
    font-weight: 800;
  }

  .value.score {
    font-size: 1.4rem;
    font-weight: 800;
  }

  .pill-group {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 0.3rem;
    align-items: stretch;
  }

  .pill-group button {
    border-radius: 999px;
    padding: 0.55rem 0.7rem;
    border: 1px solid rgba(255, 255, 255, 0.08);
    background: rgba(255, 255, 255, 0.03);
    color: #e5e7eb;
    font-weight: 700;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.3rem;
    width: 100%;
    white-space: nowrap;
    font-size: 0.82rem;
    min-height: 44px;
    transition: background 0.15s, border-color 0.15s;
  }

  .pill-group button.selected {
    background: rgba(74, 222, 128, 0.12);
    border-color: rgba(74, 222, 128, 0.5);
    color: #f8fafc;
    font-weight: 800;
  }

  .dot {
    width: 8px;
    height: 8px;
    border-radius: 999px;
    display: inline-block;
    flex-shrink: 0;
  }

  .dot.bad {
    background: #f87171;
  }

  .dot.mid {
    background: #fbbf24;
  }

  .dot.ok {
    background: #a3e635;
  }

  .dot.good {
    background: #34d399;
  }

  .pill-group button.tone-bad.selected {
    border-color: #f87171;
    background: rgba(248, 113, 113, 0.15);
  }

  .pill-group button.tone-mid.selected {
    border-color: #fbbf24;
    background: rgba(251, 191, 36, 0.15);
  }

  .pill-group button.tone-ok.selected {
    border-color: #a3e635;
    background: rgba(163, 230, 53, 0.15);
  }

  .pill-group button.tone-good.selected {
    border-color: #34d399;
    background: rgba(52, 211, 153, 0.15);
  }

  .actions {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  button.primary {
    background: linear-gradient(135deg, #4ade80, #7dd3fc);
    border: none;
    color: #0b1224;
    font-weight: 800;
    padding: 0.65rem 0.9rem;
    border-radius: 0.75rem;
    display: inline-flex;
    align-items: center;
    gap: 0.35rem;
    min-height: 44px;
    transition: opacity 0.15s;
  }

  button.primary:active {
    opacity: 0.8;
  }

  button.primary:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  button.ghost {
    background: rgba(255, 255, 255, 0.08);
    color: #f8fafc;
    border: 1px solid rgba(255, 255, 255, 0.15);
    border-radius: 0.75rem;
    padding: 0.6rem 0.9rem;
    display: inline-flex;
    gap: 0.4rem;
    align-items: center;
    min-height: 44px;
    transition: background 0.15s;
  }

  button.ghost:active {
    background: rgba(255, 255, 255, 0.15);
  }

  button.compact {
    padding: 0.45rem 0.7rem;
    font-size: 0.82rem;
  }

  textarea {
    min-height: 70px;
    resize: vertical;
  }

  .stamp {
    margin: 0;
    color: #9fb1cc;
    font-size: 0.9rem;
  }

  .stamp.small {
    font-size: 0.8rem;
    color: #8da0bf;
  }

  .badge {
    padding: 0.3rem 0.55rem;
    border-radius: 0.65rem;
    background: rgba(125, 211, 252, 0.12);
    border: 1px solid rgba(125, 211, 252, 0.25);
    color: #f8fafc;
    font-size: 0.82rem;
    display: inline-flex;
    align-items: center;
    max-width: 100%;
    white-space: nowrap;
  }

  .badge.strong {
    background: rgba(125, 211, 252, 0.16);
    border-color: rgba(125, 211, 252, 0.35);
    color: #e0f2fe;
    font-weight: 800;
    font-size: 0.85rem;
  }

  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(7, 12, 26, 0.8);
    backdrop-filter: blur(8px);
    display: grid;
    place-items: center;
    padding: 1rem;
    z-index: 20;
    animation: fadeIn 0.2s ease;
  }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  @keyframes slideUp {
    from { transform: translateY(30px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
  }

  .modal {
    background: #0f172a;
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 1rem;
    padding: 1rem;
    max-width: 980px;
    width: min(980px, 96vw);
    max-height: 90vh;
    overflow: auto;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.45);
    animation: slideUp 0.25s ease;
  }

  .modal-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 0.5rem;
  }

  .modal-header-left {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .nav-btn {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(255, 255, 255, 0.12);
    border-radius: 0.65rem;
    color: #e5e7eb;
    padding: 0.4rem;
    display: grid;
    place-items: center;
    min-height: 40px;
    min-width: 40px;
    font-size: 1.2rem;
    transition: background 0.15s;
  }

  .nav-btn:active {
    background: rgba(255, 255, 255, 0.15);
  }

  h3 {
    margin: 0 0 0.1rem 0;
    color: #f3f4f6;
    font-size: 1.1rem;
  }

  h4 {
    margin: 0 0 0.1rem 0;
    color: #f3f4f6;
    font-size: 1.05rem;
  }

  .export-card {
    background: #0f172a;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 1rem;
    padding: 1.25rem;
    width: min(460px, 96vw);
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    gap: 1rem;
    animation: slideUp 0.25s ease;
  }

  .export-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 0.75rem;
  }

  .export-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
  }

  .export-grid label {
    min-width: 0;
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
  }

  .export-grid input[type="date"] {
    width: 100%;
    min-width: 0;
    box-sizing: border-box;
    min-height: 44px;
  }

  .export-actions {
    justify-content: flex-end;
  }

  input[type="range"] {
    background: transparent;
    border: none;
    padding: 0.5rem 0;
    height: auto;
    cursor: pointer;
    accent-color: #7dd3fc;
  }

  .range-row {
    display: flex;
    align-items: center;
    gap: 0.35rem;
  }

  .range-btn {
    background: rgba(255, 255, 255, 0.08);
    border: 1px solid rgba(255, 255, 255, 0.15);
    color: #f8fafc;
    border-radius: 0.5rem;
    min-width: 36px;
    min-height: 36px;
    display: grid;
    place-items: center;
    font-size: 1rem;
    font-weight: 800;
    flex-shrink: 0;
    transition: background 0.15s;
  }

  .range-btn:active {
    background: rgba(255, 255, 255, 0.2);
  }

  .range-row input[type="range"] {
    flex: 1;
    min-width: 0;
  }

  .tabs {
    display: inline-flex;
    gap: 0.25rem;
    margin: 0.5rem 0 0.5rem 0;
    padding: 0.2rem;
    border-radius: 0.75rem;
    background: rgba(255, 255, 255, 0.02);
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
  }

  .tabs button {
    border: none;
    background: transparent;
    color: #c7d2fe;
    font-weight: 700;
    padding: 0.5rem 0.7rem;
    border-radius: 0.65rem;
    margin-bottom: 0;
    display: inline-flex;
    align-items: center;
    gap: 0.3rem;
    min-height: 40px;
    transition: background 0.2s, box-shadow 0.2s;
    position: relative;
    font-size: 0.82rem;
  }

  .tab-icon {
    font-size: 1rem;
  }

  .tab-badge {
    background: rgba(125, 211, 252, 0.3);
    color: #e0f2fe;
    font-size: 0.65rem;
    font-weight: 800;
    padding: 0.1rem 0.35rem;
    border-radius: 999px;
    min-width: 18px;
    text-align: center;
  }

  .tabs button.active {
    background: rgba(125, 211, 252, 0.2);
    color: #c7d2fe;
    box-shadow: 0 6px 18px rgba(125, 211, 252, 0.4);
    border: 1px solid rgba(125, 211, 252, 0.45);
  }

  .tabs-row {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
    flex-wrap: wrap;
  }

  .sticky-save {
    position: sticky;
    bottom: 0;
    background: linear-gradient(transparent, #0f172a 30%);
    padding: 1rem 0 0.25rem 0;
    display: flex;
    gap: 0.5rem;
    align-items: center;
    justify-content: center;
    z-index: 5;
  }

  .save-btn {
    flex: 1;
    max-width: 300px;
    justify-content: center;
    font-size: 0.95rem;
  }

  .modal-grid {
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  }

  .field-span {
    grid-column: 1 / -1;
  }

  .modal-history {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    max-height: clamp(240px, 50vh, 500px);
    overflow: auto;
    padding-right: 0.25rem;
  }

  .history-card {
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 0.85rem;
    padding: 0.5rem 0.65rem;
    background: rgba(255, 255, 255, 0.03);
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    transition: border-color 0.2s;
  }

  .history-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .note-line {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    flex-wrap: wrap;
    margin-top: 0.1rem;
  }

  .note-value {
    font-weight: 800;
    color: #f8fafc;
    font-size: 1rem;
  }

  .note-line.compact {
    flex: 1;
    min-width: 140px;
  }

  .comment {
    margin: 0.1rem 0 0 0;
    color: #cbd5e1;
    font-size: 0.85rem;
  }

  .ghost.icon {
    padding: 0.4rem 0.55rem;
    border-radius: 0.65rem;
  }

  .ghost.danger {
    border-color: rgba(248, 113, 113, 0.4);
    color: #fecdd3;
    font-size: 0.8rem;
    padding: 0.35rem 0.6rem;
  }

  .ghost.danger-confirm {
    border-color: #f87171;
    background: rgba(248, 113, 113, 0.2);
    color: #fecdd3;
    font-size: 0.8rem;
    font-weight: 800;
    padding: 0.35rem 0.6rem;
    animation: pulse 0.5s;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.6; }
  }

  .load-more {
    align-self: center;
    margin-top: 0.25rem;
  }

  .merit-field .toggle {
    margin-top: 0.25rem;
  }

  .toggle {
    width: 100%;
    padding: 0.65rem;
    border-radius: 0.85rem;
    border: 1px solid rgba(255, 255, 255, 0.12);
    background: rgba(255, 255, 255, 0.04);
    color: #e5e7eb;
    font-weight: 700;
    min-height: 44px;
    transition: background 0.15s, border-color 0.15s;
  }

  .toggle.on {
    background: rgba(74, 222, 128, 0.15);
    border-color: rgba(74, 222, 128, 0.5);
    color: #dcfce7;
  }

  .mini.muted {
    color: #9fb1cc;
    margin: 0;
  }

  .switch {
    display: inline-flex;
    align-items: center;
    gap: 0.35rem;
    font-weight: 600;
    color: #e5e7eb;
    padding: 0.35rem 0.55rem;
    border-radius: 0.65rem;
    border: 1px solid rgba(255, 255, 255, 0.12);
    background: rgba(255, 255, 255, 0.03);
    position: relative;
    cursor: pointer;
    min-height: 36px;
  }

  .switch input {
    position: absolute;
    opacity: 0;
    width: 0;
    height: 0;
    margin: 0;
    appearance: none;
  }

  .switch:focus-within {
    outline: 2px solid rgba(125, 211, 252, 0.5);
    outline-offset: 2px;
  }

  .switch.on {
    background: rgba(74, 222, 128, 0.12);
    border-color: rgba(74, 222, 128, 0.5);
    color: #dcfce7;
  }

  .switch.off {
    background: rgba(248, 113, 113, 0.12);
    border-color: rgba(248, 113, 113, 0.5);
    color: #fecdd3;
  }

  .hero-actions button,
  .actions button,
  .pill-group button,
  .student-card,
  .course-wrap button,
  .add-course-btn,
  .tabs button,
  .toggle,
  .nav-btn,
  .range-btn,
  .search-clear {
    cursor: pointer;
  }

  @media (max-width: 900px) {
    .hero {
      grid-template-columns: 1fr;
    }

    .panel-header {
      flex-direction: column;
      align-items: flex-start;
    }
  }

  @media (max-width: 600px) {
    main.page {
      padding: 0.75rem;
      gap: 0.75rem;
    }

    .modal-backdrop {
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
      align-items: stretch;
      padding: 0;
    }

    .modal {
      max-width: 100%;
      width: 100%;
      max-height: 94vh;
      border-radius: 1.25rem 1.25rem 0 0;
      padding: 0.75rem;
    }

    .export-card {
      width: 100%;
      border-radius: 1.25rem 1.25rem 0 0;
    }

    .modal-backdrop.export {
      justify-content: flex-end;
      align-items: stretch;
    }

    .tabs button {
      padding: 0.45rem 0.55rem;
      font-size: 0.78rem;
    }

    .tab-icon {
      font-size: 0.9rem;
    }

    .student-grid {
      grid-template-columns: 1fr;
    }

    .course-grid {
      grid-template-columns: repeat(2, 1fr);
    }

    .pill-group {
      grid-template-columns: repeat(2, 1fr);
    }

    .modal-grid {
      grid-template-columns: 1fr;
    }

    .sticky-save {
      padding: 0.75rem 0 0.5rem 0;
    }

    .hero-actions {
      flex-direction: column;
      align-items: stretch;
    }

    .hero-actions button {
      justify-content: center;
    }

    h1 {
      font-size: 1.4rem;
    }

    .lede {
      font-size: 0.82rem;
    }

    .student-edit-btn {
      opacity: 1;
    }
  }
</style>
