# React Hook Form - Guia Completo

## Sumário
- [Introdução](#introdução)
- [Instalação](#instalação)
- [Conceitos Básicos](#conceitos-básicos)
- [Configuração com TypeScript](#configuração-com-typescript)
- [Validação](#validação)
- [Integração com Bibliotecas de UI](#integração-com-bibliotecas-de-ui)
- [Formulários Complexos](#formulários-complexos)
- [Integração com Zod](#integração-com-zod)
- [Exemplos Práticos](#exemplos-práticos)
- [Dicas e Melhores Práticas](#dicas-e-melhores-práticas)

## Introdução

O React Hook Form é uma biblioteca que simplifica a criação e validação de formulários em React, com foco em performance e experiência do desenvolvedor. Ela utiliza hooks do React para gerenciar o estado dos formulários, reduzindo re-renderizações desnecessárias.

### Principais vantagens:
- **Performance**: Minimiza re-renderizações
- **Menos código**: Reduz a quantidade de código boilerplate
- **Fácil validação**: Integração com validadores como Yup, Zod, etc.
- **Sem dependências**: Biblioteca leve e focada

## Instalação

Para instalar o React Hook Form em seu projeto, use um dos seguintes comandos:

```bash
# NPM
npm install react-hook-form

# Yarn
yarn add react-hook-form

# PNPM
pnpm add react-hook-form
```

Para TypeScript, nenhuma instalação adicional é necessária pois o React Hook Form já vem com tipos.

## Conceitos Básicos

### Criando um formulário simples

```tsx
import React from 'react';
import { useForm } from 'react-hook-form';

type FormData = {
  name: string;
  email: string;
};

const SimpleForm = () => {
  // Inicializa o hook useForm com o tipo de dados do formulário
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>();
  
  // Função executada quando o formulário é submetido com sucesso
  const onSubmit = (data: FormData) => {
    console.log(data);
    // Aqui você pode enviar os dados para uma API
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="name">Nome</label>
        {/* register é usado para registrar campos no formulário */}
        <input id="name" {...register("name", { required: "Nome é obrigatório" })} />
        {/* Exibe mensagem de erro se houver */}
        {errors.name && <span>{errors.name.message}</span>}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input 
          id="email" 
          type="email" 
          {...register("email", { 
            required: "Email é obrigatório",
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: "Email inválido"
            }
          })} 
        />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <button type="submit">Enviar</button>
    </form>
  );
};

export default SimpleForm;
```

### Principais Hooks e Funções

- **useForm**: Hook principal que inicializa o formulário
- **register**: Registra campos do formulário
- **handleSubmit**: Processa o envio do formulário
- **formState**: Contém informações sobre o estado do formulário
- **watch**: Observa valores de campos específicos
- **setValue**: Define valores programaticamente
- **reset**: Redefine valores do formulário

## Configuração com TypeScript

O React Hook Form funciona bem com TypeScript. Definir interfaces para seus dados melhora a experiência de desenvolvimento:

```tsx
import React from 'react';
import { useForm, SubmitHandler } from 'react-hook-form';

// Definição do tipo de dados do formulário
interface IFormInput {
  firstName: string;
  lastName: string;
  age: number;
  email: string;
}

const TypedForm = () => {
  // Tipagem do useForm com a interface criada
  const { register, handleSubmit, formState: { errors } } = useForm<IFormInput>();
  
  // SubmitHandler tipado para garantir que os dados recebidos sejam do tipo correto
  const onSubmit: SubmitHandler<IFormInput> = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="firstName">Primeiro Nome</label>
        <input id="firstName" {...register("firstName", { required: true })} />
        {errors.firstName && <span>Este campo é obrigatório</span>}
      </div>
      
      <div>
        <label htmlFor="lastName">Sobrenome</label>
        <input id="lastName" {...register("lastName", { required: true })} />
        {errors.lastName && <span>Este campo é obrigatório</span>}
      </div>
      
      <div>
        <label htmlFor="age">Idade</label>
        {/* Observe o valueAsNumber para converter string para número */}
        <input 
          id="age" 
          type="number" 
          {...register("age", { 
            valueAsNumber: true, 
            required: true,
            min: {
              value: 18,
              message: "Você deve ter pelo menos 18 anos"
            }
          })} 
        />
        {errors.age && <span>{errors.age.message}</span>}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input 
          id="email" 
          type="email" 
          {...register("email", { 
            required: "Email é obrigatório",
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: "Email inválido"
            }
          })} 
        />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <button type="submit">Enviar</button>
    </form>
  );
};

export default TypedForm;
```

## Validação

O React Hook Form oferece várias maneiras de validar formulários:

### Validação básica

```tsx
// Validação de campo obrigatório
<input {...register("name", { required: "Nome é obrigatório" })} />

// Validação de comprimento mínimo
<input {...register("password", { 
  minLength: {
    value: 8,
    message: "A senha deve ter pelo menos 8 caracteres"
  }
})} />

// Validação com expressão regular
<input {...register("email", { 
  pattern: {
    value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
    message: "Email inválido"
  }
})} />
```

### Validação personalizada

```tsx
<input {...register("username", {
  validate: {
    notAdmin: (value) => value !== "admin" || "Este username não está disponível",
    notBlackListed: (value) => !["test", "user", "demo"].includes(value) || "Este username não está disponível",
    checkAvailability: async (value) => {
      // Verifica disponibilidade do username em uma API
      const response = await fetch(`/api/check-username?username=${value}`);
      const data = await response.json();
      return data.available || "Este username já está em uso";
    }
  }
})} />
```

## Integração com Bibliotecas de UI

O React Hook Form se integra facilmente com bibliotecas de UI como Material-UI, Chakra UI, ou Tailwind.

### Exemplo com Chakra UI

Primeiro, instale as dependências:

```bash
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

Em seguida, crie um formulário com Chakra UI:

```tsx
import React from 'react';
import { useForm, Controller } from 'react-hook-form';
import {
  FormControl,
  FormLabel,
  FormErrorMessage,
  Input,
  Button,
  VStack,
  Box,
} from '@chakra-ui/react';

interface FormData {
  name: string;
  email: string;
}

const ChakraForm = () => {
  const { register, handleSubmit, control, formState: { errors } } = useForm<FormData>();
  
  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <Box maxW="md" mx="auto" mt={8}>
      <form onSubmit={handleSubmit(onSubmit)}>
        <VStack spacing={4} align="flex-start">
          <FormControl isInvalid={!!errors.name}>
            <FormLabel htmlFor="name">Nome</FormLabel>
            <Input 
              id="name"
              placeholder="Seu nome"
              {...register("name", { required: "Nome é obrigatório" })}
            />
            <FormErrorMessage>{errors.name?.message}</FormErrorMessage>
          </FormControl>

          <FormControl isInvalid={!!errors.email}>
            <FormLabel htmlFor="email">Email</FormLabel>
            {/* Usando Controller para componentes mais complexos */}
            <Controller
              name="email"
              control={control}
              rules={{ 
                required: "Email é obrigatório",
                pattern: {
                  value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                  message: "Email inválido"
                }
              }}
              render={({ field }) => (
                <Input 
                  id="email"
                  placeholder="seu-email@exemplo.com"
                  {...field}
                />
              )}
            />
            <FormErrorMessage>{errors.email?.message}</FormErrorMessage>
          </FormControl>

          <Button type="submit" colorScheme="blue" alignSelf="flex-end">
            Enviar
          </Button>
        </VStack>
      </form>
    </Box>
  );
};

export default ChakraForm;
```

## Formulários Complexos

### Campos Dinâmicos

Para criar campos dinâmicos, use o hook `useFieldArray`:

```tsx
import React from 'react';
import { useForm, useFieldArray } from 'react-hook-form';

interface PhoneInput {
  type: string;
  number: string;
}

interface FormData {
  name: string;
  phoneNumbers: PhoneInput[];
}

const DynamicForm = () => {
  const { register, control, handleSubmit, formState: { errors } } = useForm<FormData>({
    defaultValues: {
      name: '',
      phoneNumbers: [{ type: 'home', number: '' }]
    }
  });
  
  // Inicializa o array de campos dinâmicos
  const { fields, append, remove } = useFieldArray({
    control,
    name: "phoneNumbers"
  });
  
  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="name">Nome</label>
        <input id="name" {...register("name", { required: "Nome é obrigatório" })} />
        {errors.name && <span>{errors.name.message}</span>}
      </div>
      
      <div>
        <h3>Telefones</h3>
        {fields.map((field, index) => (
          <div key={field.id}>
            <select {...register(`phoneNumbers.${index}.type` as const)}>
              <option value="home">Casa</option>
              <option value="work">Trabalho</option>
              <option value="mobile">Celular</option>
            </select>
            
            <input
              type="tel"
              {...register(`phoneNumbers.${index}.number` as const, {
                required: "Número de telefone é obrigatório"
              })}
            />
            
            <button type="button" onClick={() => remove(index)}>
              Remover
            </button>
            
            {errors.phoneNumbers?.[index]?.number && (
              <span>{errors.phoneNumbers[index]?.number?.message}</span>
            )}
          </div>
        ))}
        
        <button
          type="button"
          onClick={() => append({ type: 'home', number: '' })}
        >
          Adicionar Telefone
        </button>
      </div>
      
      <button type="submit">Enviar</button>
    </form>
  );
};

export default DynamicForm;
```

### Formulários com Múltiplos Passos

```tsx
import React, { useState } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';

interface FormData {
  // Passo 1
  firstName: string;
  lastName: string;
  
  // Passo 2
  email: string;
  phone: string;
  
  // Passo 3
  address: string;
  city: string;
  zipCode: string;
}

// Componente para o passo 1
const Step1 = () => {
  const { register, formState: { errors } } = useFormContext<FormData>();
  
  return (
    <div>
      <h2>Informações Pessoais</h2>
      
      <div>
        <label htmlFor="firstName">Primeiro Nome</label>
        <input 
          id="firstName" 
          {...register("firstName", { required: "Primeiro nome é obrigatório" })} 
        />
        {errors.firstName && <span>{errors.firstName.message}</span>}
      </div>
      
      <div>
        <label htmlFor="lastName">Sobrenome</label>
        <input 
          id="lastName" 
          {...register("lastName", { required: "Sobrenome é obrigatório" })} 
        />
        {errors.lastName && <span>{errors.lastName.message}</span>}
      </div>
    </div>
  );
};

// Componente para o passo 2
const Step2 = () => {
  const { register, formState: { errors } } = useFormContext<FormData>();
  
  return (
    <div>
      <h2>Informações de Contato</h2>
      
      <div>
        <label htmlFor="email">Email</label>
        <input 
          id="email" 
          type="email" 
          {...register("email", { 
            required: "Email é obrigatório",
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: "Email inválido"
            }})} 
        />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <label htmlFor="phone">Telefone</label>
        <input 
          id="phone" 
          {...register("phone", { required: "Telefone é obrigatório" })} 
        />
        {errors.phone && <span>{errors.phone.message}</span>}
      </div>
    </div>
  );
};

// Componente para o passo 3
const Step3 = () => {
  const { register, formState: { errors } } = useFormContext<FormData>();
  
  return (
    <div>
      <h2>Endereço</h2>
      
      <div>
        <label htmlFor="address">Rua</label>
        <input 
          id="address" 
          {...register("address", { required: "Endereço é obrigatório" })} 
        />
        {errors.address && <span>{errors.address.message}</span>}
      </div>
      
      <div>
        <label htmlFor="city">Cidade</label>
        <input 
          id="city" 
          {...register("city", { required: "Cidade é obrigatória" })} 
        />
        {errors.city && <span>{errors.city.message}</span>}
      </div>
      
      <div>
        <label htmlFor="zipCode">CEP</label>
        <input 
          id="zipCode" 
          {...register("zipCode", { required: "CEP é obrigatório" })} 
        />
        {errors.zipCode && <span>{errors.zipCode.message}</span>}
      </div>
    </div>
  );
};

// Componente principal
const MultiStepForm = () => {
  const [step, setStep] = useState(1);
  const methods = useForm<FormData>({
    mode: "onBlur", // Valida quando o campo perde o foco
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      phone: '',
      address: '',
      city: '',
      zipCode: ''
    }
  });
  
  const { handleSubmit, trigger } = methods;
  
  // Função para avançar para o próximo passo
  const nextStep = async () => {
    let fieldsToValidate: (keyof FormData)[] = [];
    
    // Define quais campos validar com base no passo atual
    if (step === 1) {
      fieldsToValidate = ['firstName', 'lastName'];
    } else if (step === 2) {
      fieldsToValidate = ['email', 'phone'];
    }
    
    // Valida os campos do passo atual
    const isValid = await trigger(fieldsToValidate);
    
    if (isValid) {
      setStep(step + 1);
    }
  };
  
  // Função para voltar ao passo anterior
  const prevStep = () => {
    setStep(step - 1);
  };
  
  // Função executada quando o formulário é enviado
  const onSubmit = (data: FormData) => {
    console.log('Formulário enviado:', data);
    // Aqui você enviaria os dados para o servidor
  };
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={handleSubmit(onSubmit)}>
        {step === 1 && <Step1 />}
        {step === 2 && <Step2 />}
        {step === 3 && <Step3 />}
        
        <div style={{ marginTop: '20px' }}>
          {step > 1 && (
            <button type="button" onClick={prevStep}>
              Anterior
            </button>
          )}
          
          {step < 3 ? (
            <button type="button" onClick={nextStep}>
              Próximo
            </button>
          ) : (
            <button type="submit">
              Enviar
            </button>
          )}
        </div>
      </form>
    </FormProvider>
  );
};

export default MultiStepForm;
```

## Integração com Zod

Zod é uma biblioteca de validação de esquemas baseada em TypeScript que funciona muito bem com React Hook Form. Primeiro, instale-a:

```bash
npm install zod @hookform/resolvers
```

Exemplo de uso:

```tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Definição do esquema de validação com Zod
const userSchema = z.object({
  username: z.string()
    .min(3, { message: "O username deve ter pelo menos 3 caracteres" })
    .max(20, { message: "O username deve ter no máximo 20 caracteres" }),
  email: z.string()
    .email({ message: "Email inválido" }),
  password: z.string()
    .min(8, { message: "A senha deve ter pelo menos 8 caracteres" })
    .regex(/[A-Z]/, { message: "A senha deve conter pelo menos uma letra maiúscula" })
    .regex(/[0-9]/, { message: "A senha deve conter pelo menos um número" }),
  confirmPassword: z.string()
}).refine((data) => data.password === data.confirmPassword, {
  message: "As senhas não coincidem",
  path: ["confirmPassword"],
});

// Tipo inferido do esquema Zod
type UserFormData = z.infer<typeof userSchema>;

const ZodForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<UserFormData>({
    resolver: zodResolver(userSchema)
  });
  
  const onSubmit = (data: UserFormData) => {
    console.log(data);
    // Processamento dos dados do formulário...
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="username">Username</label>
        <input id="username" {...register("username")} />
        {errors.username && <span>{errors.username.message}</span>}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register("email")} />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <label htmlFor="password">Senha</label>
        <input id="password" type="password" {...register("password")} />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      <div>
        <label htmlFor="confirmPassword">Confirmar Senha</label>
        <input id="confirmPassword" type="password" {...register("confirmPassword")} />
        {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}
      </div>
      
      <button type="submit">Cadastrar</button>
    </form>
  );
};

export default ZodForm;
```

## Exemplos Práticos

### Formulário de Login

```tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

// Esquema de validação
const loginSchema = z.object({
  email: z.string()
    .email({ message: "Email inválido" })
    .min(1, { message: "Email é obrigatório" }),
  password: z.string()
    .min(1, { message: "Senha é obrigatória" }),
  rememberMe: z.boolean().optional()
});

type LoginFormData = z.infer<typeof loginSchema>;

const LoginForm = () => {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
      rememberMe: false
    }
  });
  
  const onSubmit = async (data: LoginFormData) => {
    try {
      // Simulação de request para API
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Login submetido:', data);
      // Aqui você chamaria sua API de autenticação
    } catch (error) {
      console.error('Erro ao fazer login:', error);
    }
  };

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded shadow">
      <h2 className="text-2xl font-bold mb-6">Login</h2>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        <div className="mb-4">
          <label 
            htmlFor="email" 
            className="block text-gray-700 mb-2"
          >
            Email
          </label>
          <input
            id="email"
            type="email"
            className={`w-full p-2 border rounded ${errors.email ? 'border-red-500' : 'border-gray-300'}`}
            placeholder="seu-email@exemplo.com"
            {...register("email")}
          />
          {errors.email && (
            <span className="text-red-500 text-sm mt-1">
              {errors.email.message}
            </span>
          )}
        </div>
        
        <div className="mb-4">
          <label 
            htmlFor="password" 
            className="block text-gray-700 mb-2"
          >
            Senha
          </label>
          <input
            id="password"
            type="password"
            className={`w-full p-2 border rounded ${errors.password ? 'border-red-500' : 'border-gray-300'}`}
            placeholder="Sua senha"
            {...register("password")}
          />
          {errors.password && (
            <span className="text-red-500 text-sm mt-1">
              {errors.password.message}
            </span>
          )}
        </div>
        
        <div className="mb-6">
          <label className="flex items-center">
            <input
              type="checkbox"
              className="mr-2"
              {...register("rememberMe")}
            />
            <span className="text-gray-700">Lembrar de mim</span>
          </label>
        </div>
        
        <button
          type="submit"
          className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-300"
          disabled={isSubmitting}
        >
          {isSubmitting ? 'Processando...' : 'Entrar'}
        </button>
      </form>
    </div>
  );
};

export default LoginForm;
```

### Formulário de Cadastro de Produtos

```tsx
import React from 'react';
import { useForm, useFieldArray } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

// Esquema para variante do produto
const variantSchema = z.object({
  name: z.string().min(1, { message: "Nome da variante é obrigatório" }),
  price: z.coerce.number().min(0.01, { message: "Preço deve ser maior que zero" }),
  sku: z.string().min(1, { message: "SKU é obrigatório" })
});

// Esquema para produto
const productSchema = z.object({
  name: z.string().min(3, { message: "Nome do produto deve ter pelo menos 3 caracteres" }),
  description: z.string().min(10, { message: "Descrição deve ter pelo menos 10 caracteres" }),
  category: z.string().min(1, { message: "Categoria é obrigatória" }),
  variants: z.array(variantSchema).min(1, { message: "Pelo menos uma variante é necessária" })
});

type ProductFormData = z.infer<typeof productSchema>;

const ProductForm = () => {
  const { 
    register, 
    control, 
    handleSubmit, 
    formState: { errors, isSubmitting } 
  } = useForm<ProductFormData>({
    resolver: zodResolver(productSchema),
    defaultValues: {
      name: '',
      description: '',
      category: '',
      variants: [{ name: '', price: 0, sku: '' }]
    }
  });
  
  const { fields, append, remove } = useFieldArray({
    control,
    name: "variants"
  });
  
  const onSubmit = async (data: ProductFormData) => {
    try {
      // Simulação de request para API
      await new Promise(resolve => setTimeout(resolve, 1500));
      console.log('Produto submetido:', data);
      // Aqui você chamaria sua API para salvar o produto
    } catch (error) {
      console.error('Erro ao cadastrar produto:', error);
    }
  };

  return (
    <div className="max-w-2xl mx-auto p-6 bg-white rounded shadow">
      <h2 className="text-2xl font-bold mb-6">Cadastro de Produto</h2>
      
      <form onSubmit={handleSubmit(onSubmit)}>
        <div className="mb-4">
          <label htmlFor="name" className="block text-gray-700 mb-2">
            Nome do Produto
          </label>
          <input
            id="name"
            className={`w-full p-2 border rounded ${errors.name ? 'border-red-500' : 'border-gray-300'}`}
            {...register("name")}
          />
          {errors.name && (
            <span className="text-red-500 text-sm mt-1">
              {errors.name.message}
            </span>
          )}
        </div>
        
        <div className="mb-4">
          <label htmlFor="description" className="block text-gray-700 mb-2">
            Descrição
          </label>
          <textarea
            id="description"
            rows={4}
            className={`w-full p-2 border rounded ${errors.description ? 'border-red-500' : 'border-gray-300'}`}
            {...register("description")}
          />
          {errors.description && (
            <span className="text-red-500 text-sm mt-1">
              {errors.description.message}
            </span>
          )}
        </div>
        
        <div className="mb-4">
          <label htmlFor="category" className="block text-gray-700 mb-2">
            Categoria
          </label>
          <select
            id="category"
            className={`w-full p-2 border rounded ${errors.category ? 'border-red-500' : 'border-gray-300'}`}
            {...register("category")}
          >
            <option value="">Selecione uma categoria</option>
            <option value="electronics">Eletrônicos</option>
            <option value="clothing">Roupas</option>
            <option value="books">Livros</option>
            <option value="home">Casa e Decoração</option>
          </select>
          {errors.category && (
            <span className="text-red-500 text-sm mt-1">
              {errors.category.message}
            </span>
          )}
        </div>
        
        <div className="mb-4">
          <h3 className="text-lg font-medium mb-2">Variantes do Produto</h3>
          {errors.variants?.message && (
            <span className="text-red-500 text-sm block mb-2">
              {errors.variants.message}
            </span>
          )}
          
          {fields.map((field, index) => (
            <div key={field.id} className="mb-4 p-4 border rounded bg-gray-50">
              <div className="flex justify-between mb-2">
                <h4 className="font-medium">Variante {index + 1}</h4>
                {fields.length > 1 && (
                  <button 
                    type="button"
                    className="text-red-500 hover:text-red-700"
                    onClick={() => remove(index)}
                  >
                    Remover
                  </button>
                )}
              </div>
              
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <label htmlFor={`variants.${index}.name`} className="block text-gray-700 mb-1">
                    Nome
                  </label>
                  <input
                    id={`variants.${index}.name`}
                    className={`w-full p-2 border rounded ${errors.variants?.[index]?.name ? 'border-red-500' : 'border-gray-300'}`}
                    {...register(`variants.${index}.name` as const)}
                  />
                  {errors.variants?.[index]?.name && (
                    <span className="text-red-500 text-sm mt-1">
                      {errors.variants[index]?.name?.message}
                    </span>
                  )}
                </div>
                
                <div>
                  <label htmlFor={`variants.${index}.price`} className="block text-gray-700 mb-1">
                    Preço
                  </label>
                  <input
                    id={`variants.${index}.price`}
                    type="number"
                    step="0.01"
                    className={`w-full p-2 border rounded ${errors.variants?.[index]?.price ? 'border-red-500' : 'border-gray-300'}`}
                    {...register(`variants.${index}.price` as const)}
                  />
                  {errors.variants?.[index]?.price && (
                    <span className="text-red-500 text-sm mt-1">
                      {errors.variants[index]?.price?.message}
                    </span>
                  )}
                </div>
                
                <div>
                  <label htmlFor={`variants.${index}.sku`} className="block text-gray-700 mb-1">
                    SKU
                  </label>
                  <input
                    id={`variants.${index}.sku`}
                    className={`w-full p-2 border rounded ${errors.variants?.[index]?.sku ? 'border-red-500' : 'border-gray-300'}`}
                    {...register(`variants.${index}.sku` as const)}
                  />
                  {errors.variants?.[index]?.sku && (
                    <span className="text-red-500 text-sm mt-1">
                      {errors.variants[index]?.sku?.message}
                    </span>
                  )}
                </div>
              </div>
            </div>
          ))}
          
          <button
            type="button"
            className="mb-6 p-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300"
            onClick={() => append({ name: '', price: 0, sku: '' })}
          >
            + Adicionar Variante
          </button>
        </div>
        
        <button
          type="submit"
          className="w-full bg-blue-500 text-white p-3 rounded hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-300"
          disabled={isSubmitting}
        >
          {isSubmitting ? 'Salvando...' : 'Salvar Produto'}
        </button>
      </form>
    </div>
  );
};

export default ProductForm;
```

## Dicas e Melhores Práticas

### Organização de Código

Para projetos maiores, é recomendável separar a lógica de validação dos componentes:

```tsx
// src/schemas/userSchema.ts
import { z } from 'zod';

export const userSchema = z.object({
  name: z.string().min(1, { message: "Nome é obrigatório" }),
  email: z.string().email({ message: "Email inválido" }),
  password: z.string().min(8, { message: "Senha deve ter no mínimo 8 caracteres" })
});

export type UserFormData = z.infer<typeof userSchema>;
```

```tsx
// src/hooks/useUserForm.ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { userSchema, UserFormData } from '../schemas/userSchema';

export const useUserForm = (defaultValues?: Partial<UserFormData>) => {
  return useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
      ...defaultValues
    }
  });
};
```

```tsx
// src/components/UserForm.tsx
import React from 'react';
import { useUserForm } from '../hooks/useUserForm';

const UserForm = () => {
  const { register, handleSubmit, formState: { errors } } = useUserForm();
  
  const onSubmit = (data: UserFormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Campos do formulário */}
    </form>
  );
};
```

### Performance

- **Evite re-renderizações desnecessárias**: Componentes com formulários complexos podem se beneficiar da memorização (memo) e do uso de contextos para evitar re-renderizações.
- **Validação no momento certo**: Use os modos de validação apropriados:
  - `onChange`: Valida em tempo real (pode ser custoso para formulários complexos)
  - `onBlur`: Valida quando o campo perde o foco
  - `onSubmit`: Valida apenas quando o formulário é enviado
  - `onTouched`: Valida quando o campo é tocado (clicado) e depois quando muda
  
```tsx
const { register, handleSubmit } = useForm({
  mode: 'onBlur', // Valida quando o campo perde o foco
  reValidateMode: 'onChange' // Re-valida quando o valor muda após primeira validação
});
```

### Debug

Para depurar formulários complexos, use o hook `useWatch` para monitorar valores específicos:

```tsx
import { useForm, useWatch } from 'react-hook-form';

const MyForm = () => {
  const { control, register } = useForm();
  
  // Monitora o valor do campo "email"
  const email = useWatch({
    control,
    name: "email",
    defaultValue: ""
  });
  
  console.log("Email atual:", email);
  
  return (
    <form>
      <input {...register("email")} />
    </form>
  );
};
```

Ou use o componente `DevTool` para uma depuração visual em desenvolvimento:

```tsx
import { useForm } from 'react-hook-form';
import { DevTool } from '@hookform/devtools';

const MyForm = () => {
  const { control, register } = useForm();
  
  return (
    <>
      <form>
        <input {...register("email")} />
      </form>
      {process.env.NODE_ENV === 'development' && <DevTool control={control} />}
    </>
  );
};
```

### Acessibilidade

- **Associe labels com campos**: Sempre use o atributo `htmlFor` em labels e `id` em inputs.
- **Mensagens de erro**: Use o atributo `aria-invalid` para indicar campos inválidos e `aria-describedby` para mensagens de erro.

```tsx
<div>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    aria-invalid={errors.email ? "true" : "false"}
    aria-describedby={errors.email ? "email-error" : undefined}
    {...register("email", { required: "Email é obrigatório" })}
  />
  {errors.email && (
    <span id="email-error" role="alert">
      {errors.email.message}
    </span>
  )}
</div>
```

### Integração com APIs

Para formulários que enviam dados para APIs:

```tsx
import { useForm } from 'react-hook-form';
import { useState } from 'react';

const ApiForm = () => {
  const { register, handleSubmit, formState: { errors, isSubmitting }, reset } = useForm();
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [apiError, setApiError] = useState<string | null>(null);
  
  const onSubmit = async (data: any) => {
    try {
      setSubmitStatus('loading');
      setApiError(null);
      
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      });
      
      const result = await response.json();
      
      if (!response.ok) {
        throw new Error(result.message || 'Ocorreu um erro ao enviar o formulário');
      }
      
      setSubmitStatus('success');
      reset(); // Limpa o formulário após envio bem-sucedido
    } catch (error) {
      setSubmitStatus('error');
      setApiError(error instanceof Error ? error.message : 'Ocorreu um erro desconhecido');
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Campos do formulário */}
      
      {submitStatus === 'error' && (
        <div className="p-3 bg-red-100 text-red-700 rounded mb-4">
          {apiError || 'Ocorreu um erro ao enviar o formulário. Tente novamente.'}
        </div>
      )}
      
      {submitStatus === 'success' && (
        <div className="p-3 bg-green-100 text-green-700 rounded mb-4">
          Formulário enviado com sucesso!
        </div>
      )}
      
      <button 
        type="submit" 
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600 disabled:bg-gray-400"
      >
        {isSubmitting ? 'Enviando...' : 'Enviar'}
      </button>
    </form>
  );
};
```

## Conclusão

O React Hook Form é uma biblioteca poderosa e flexível para gerenciamento de formulários em React. Suas principais vantagens são:

1. **Performance**: Minimiza re-renderizações desnecessárias.
2. **Validação robusta**: Integração com Zod, Yup e outras bibliotecas de validação.
3. **Tipagem**: Excelente suporte para TypeScript.
4. **Flexibilidade**: Funciona com qualquer biblioteca de UI.
5. **Manutenibilidade**: Reduz o código boilerplate e melhora a organização.

Ao seguir as boas práticas e padrões demonstrados neste guia, você poderá criar formulários robustos, acessíveis e fáceis de manter em seus projetos React.
