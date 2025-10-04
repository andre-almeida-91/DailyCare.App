# Frontend Administrativo - DailyCare

## 📱 Visão Geral

O painel administrativo do DailyCare é uma aplicação web para gestão de escolas, turmas, educadores e configurações do sistema. É utilizado por administradores de escolas e super administradores.

---

## 🎯 Funcionalidades Principais

### Para Administradores de Escola
- Gestão de turmas e crianças
- Gestão de educadores
- Registo de atividades diárias
- Visualização de relatórios
- Configurações da escola
- Gestão de pais e acessos

### Para Super Administradores
- Gestão de múltiplas escolas
- Provisionamento de tenants
- Controlo de acessos globais
- Monitorização do sistema
- Gestão de licenças

---

## 🏗️ Estrutura do Projeto

```
admin-app/
├── src/
│   ├── features/
│   │   ├── schools/            # Gestão de escolas
│   │   ├── classes/            # Gestão de turmas
│   │   ├── educators/          # Gestão de educadores
│   │   ├── activities/         # Registo de atividades
│   │   ├── children/           # Gestão de crianças
│   │   ├── reports/            # Relatórios administrativos
│   │   └── settings/           # Configurações
│   ├── components/
│   │   ├── admin/              # Componentes específicos admin
│   │   ├── tables/             # Tabelas de dados
│   │   └── forms/              # Formulários complexos
│   └── layouts/
│       └── AdminLayout.tsx     # Layout com sidebar
```

---

## 🎨 Layout Administrativo

### Sidebar de Navegação

```typescript
const menuItems = [
  {
    title: 'Dashboard',
    icon: <DashboardIcon />,
    path: '/admin/dashboard',
  },
  {
    title: 'Escolas',
    icon: <SchoolIcon />,
    path: '/admin/schools',
    permission: 'manage_schools',
  },
  {
    title: 'Turmas',
    icon: <ClassIcon />,
    path: '/admin/classes',
  },
  {
    title: 'Crianças',
    icon: <ChildCareIcon />,
    path: '/admin/children',
  },
  {
    title: 'Educadores',
    icon: <PeopleIcon />,
    path: '/admin/educators',
  },
  {
    title: 'Atividades',
    icon: <EventIcon />,
    path: '/admin/activities',
  },
  {
    title: 'Relatórios',
    icon: <AssessmentIcon />,
    path: '/admin/reports',
  },
  {
    title: 'Configurações',
    icon: <SettingsIcon />,
    path: '/admin/settings',
  },
];
```

---

## 📊 Páginas Principais

### 1. Dashboard Administrativo

Visão geral com métricas e estatísticas:

```typescript
export const AdminDashboard: FC = () => {
  return (
    <Box>
      <Grid container spacing={3}>
        {/* KPIs */}
        <Grid item xs={12} md={3}>
          <StatCard
            title="Total de Crianças"
            value={145}
            icon={<ChildCareIcon />}
            trend="+5.2%"
          />
        </Grid>
        
        {/* Gráficos */}
        <Grid item xs={12} md={8}>
          <Card>
            <CardContent>
              <Typography variant="h6">Atividades por Dia</Typography>
              <LineChart data={activitiesData} />
            </CardContent>
          </Card>
        </Grid>
      </Grid>
    </Box>
  );
};
```

### 2. Gestão de Escolas (Super Admin)

```typescript
export const SchoolsPage: FC = () => {
  return (
    <>
      <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 3 }}>
        <Typography variant="h4">Gestão de Escolas</Typography>
        <Button
          variant="contained"
          startIcon={<AddIcon />}
          onClick={handleCreateSchool}
        >
          Nova Escola
        </Button>
      </Box>

      <DataGrid
        rows={schools}
        columns={[
          { field: 'name', headerName: 'Nome', width: 200 },
          { field: 'code', headerName: 'Código', width: 100 },
          { field: 'totalChildren', headerName: 'Crianças', width: 100 },
          { field: 'isActive', headerName: 'Ativo', width: 100 },
          {
            field: 'actions',
            headerName: 'Ações',
            width: 150,
            renderCell: (params) => (
              <Actions schoolId={params.row.id} />
            ),
          },
        ]}
      />
    </>
  );
};
```

### 3. Gestão de Turmas

```typescript
export const ClassesPage: FC = () => {
  const [selectedClass, setSelectedClass] = useState<Class | null>(null);
  
  return (
    <Box>
      <ClassList onSelectClass={setSelectedClass} />
      
      {selectedClass && (
        <Drawer open={!!selectedClass} onClose={() => setSelectedClass(null)}>
          <ClassDetails classId={selectedClass.id} />
        </Drawer>
      )}
    </Box>
  );
};
```

### 4. Registo Rápido de Atividades

Interface otimizada para educadores:

```typescript
export const QuickActivityRegistration: FC = () => {
  const [selectedChildren, setSelectedChildren] = useState<string[]>([]);
  const [activityType, setActivityType] = useState<ActivityType>('feeding');

  return (
    <Card>
      <CardContent>
        {/* Seleção de crianças */}
        <ChildrenSelector
          value={selectedChildren}
          onChange={setSelectedChildren}
        />

        {/* Tabs de tipos de atividade */}
        <Tabs value={activityType} onChange={(_, v) => setActivityType(v)}>
          <Tab label="Alimentação" value="feeding" />
          <Tab label="Sono" value="sleep" />
          <Tab label="Fralda" value="diaper" />
        </Tabs>

        {/* Formulário específico */}
        {activityType === 'feeding' && <FeedingForm />}
        {activityType === 'sleep' && <SleepForm />}
        {activityType === 'diaper' && <DiaperForm />}

        {/* Ações */}
        <Button
          variant="contained"
          onClick={handleBulkRegister}
          disabled={selectedChildren.length === 0}
        >
          Registar para {selectedChildren.length} crianças
        </Button>
      </CardContent>
    </Card>
  );
};
```

---

## 🔐 Controlo de Permissões

```typescript
// Hook para verificar permissões
export const usePermissions = () => {
  const { user } = useAuth();

  const hasPermission = (permission: string): boolean => {
    return user?.permissions.includes(permission) || false;
  };

  const hasRole = (role: string): boolean => {
    return user?.role === role;
  };

  return { hasPermission, hasRole };
};

// Componente de proteção
export const ProtectedRoute: FC<ProtectedRouteProps> = ({
  children,
  permission,
  role,
}) => {
  const { hasPermission, hasRole } = usePermissions();

  if (permission && !hasPermission(permission)) {
    return <Navigate to="/admin/unauthorized" />;
  }

  if (role && !hasRole(role)) {
    return <Navigate to="/admin/unauthorized" />;
  }

  return <>{children}</>;
};
```

---

## 📱 Componentes Específicos

### Data Grid Customizada

```typescript
export const AdminDataGrid: FC<AdminDataGridProps> = ({
  rows,
  columns,
  loading,
  onRowClick,
}) => {
  return (
    <DataGrid
      rows={rows}
      columns={columns}
      loading={loading}
      autoHeight
      pagination
      paginationMode="server"
      onRowClick={onRowClick}
      localeText={ptPTLocale}
      sx={{
        '& .MuiDataGrid-cell:hover': {
          cursor: 'pointer',
        },
      }}
    />
  );
};
```

### Formulário Multi-Step

```typescript
export const MultiStepForm: FC<MultiStepFormProps> = ({
  steps,
  onComplete,
}) => {
  const [activeStep, setActiveStep] = useState(0);

  return (
    <Box>
      <Stepper activeStep={activeStep}>
        {steps.map((step) => (
          <Step key={step.label}>
            <StepLabel>{step.label}</StepLabel>
          </Step>
        ))}
      </Stepper>

      <Box sx={{ mt: 3 }}>
        {steps[activeStep].component}
      </Box>

      <Box sx={{ display: 'flex', justifyContent: 'space-between', mt: 3 }}>
        <Button
          disabled={activeStep === 0}
          onClick={() => setActiveStep((s) => s - 1)}
        >
          Anterior
        </Button>
        <Button
          variant="contained"
          onClick={() =>
            activeStep === steps.length - 1
              ? onComplete()
              : setActiveStep((s) => s + 1)
          }
        >
          {activeStep === steps.length - 1 ? 'Concluir' : 'Próximo'}
        </Button>
      </Box>
    </Box>
  );
};
```

---

## 📊 Relatórios Administrativos

### Relatório de Frequência

```typescript
export const AttendanceReport: FC = () => {
  const [dateRange, setDateRange] = useState<DateRange>({
    start: startOfMonth(new Date()),
    end: endOfMonth(new Date()),
  });

  return (
    <Card>
      <CardContent>
        <Box sx={{ display: 'flex', gap: 2, mb: 3 }}>
          <DateRangePicker
            value={dateRange}
            onChange={setDateRange}
          />
          <Button
            variant="outlined"
            startIcon={<DownloadIcon />}
            onClick={handleExport}
          >
            Exportar PDF
          </Button>
        </Box>

        <DataGrid
          rows={attendanceData}
          columns={[
            { field: 'childName', headerName: 'Criança' },
            { field: 'totalDays', headerName: 'Dias Presentes' },
            { field: 'absences', headerName: 'Faltas' },
            { field: 'percentage', headerName: 'Percentagem' },
          ]}
        />
      </CardContent>
    </Card>
  );
};
```

---

## 🎨 Tema Administrativo

```typescript
const adminTheme = createTheme({
  palette: {
    mode: 'light',
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
    background: {
      default: '#f5f5f5',
      paper: '#ffffff',
    },
  },
  components: {
    MuiDrawer: {
      styleOverrides: {
        paper: {
          width: 280,
          backgroundColor: '#1e293b',
          color: '#ffffff',
        },
      },
    },
  },
});
```

---

## 🔄 Estado Global (Redux)

```typescript
// Admin Slice
const adminSlice = createSlice({
  name: 'admin',
  initialState: {
    selectedSchool: null,
    selectedClass: null,
    filters: {},
  },
  reducers: {
    setSelectedSchool: (state, action) => {
      state.selectedSchool = action.payload;
    },
    setSelectedClass: (state, action) => {
      state.selectedClass = action.payload;
    },
  },
});
```

---

## 📝 Formulários Avançados

### Formulário de Criação de Escola

```typescript
const schoolSchema = z.object({
  name: z.string().min(3, 'Nome deve ter no mínimo 3 caracteres'),
  code: z.string().regex(/^[A-Z0-9]+$/, 'Código inválido'),
  email: z.string().email('Email inválido'),
  phoneNumber: z.string().regex(/^\+351\d{9}$/, 'Telefone inválido'),
  address: z.object({
    street: z.string(),
    postalCode: z.string(),
    city: z.string(),
  }),
});

export const CreateSchoolForm: FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<SchoolFormData>({
    resolver: zodResolver(schoolSchema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <TextField
        {...register('name')}
        label="Nome da Escola"
        error={!!errors.name}
        helperText={errors.name?.message}
      />
      {/* Mais campos... */}
    </form>
  );
};
```

---

**Anterior**: [Frontend Pais](./04-FRONTEND-PAIS.md)  
**Próximo**: [Backend Microserviços](./06-BACKEND-MICROSERVICOS.md)  
**Voltar ao**: [Índice](./00-INDICE.md)
