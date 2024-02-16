**step 1** creating an student model in prisma 
```
model Student{
    id        Int      @id @default(autoincrement())
    name  String
    gender String
    department String
    category String
    batch Int
    address String
}
```
**step 2** creating an student form in form.tsx sed
```
step 3 => const StudentForm = () => {

  const formSchema = z.object({
    name: z.string(),
    department: z.string(),
    gender: z.string(),
    category: z.string(),
    batch: z.string(),
    address: z.string(),

  });

  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      name: "",
      department: "",
      gender: "",
      category: "",
      batch: "",
      address: "",
    },
  });
```
**step 4** : getting the form form shad cn
**step 5** : creating an student router in src/server/api/routers/student.ts
```
// this is the student router for the student model man to fetch the student detail from the form to the backend
import {z} from "zod";

  
import {  createTRPCRouter,// protectedProcedure, publicProcedure,} from "@/server/api/trpc";

  export const studentRouter = createTRPCRouter({
    findMany: publicProcedure.query(async ({ ctx }) => {
      const students = await ctx.db.student.findMany({
      });
      return students;
    }),
});
```

**step 6** : after creating the studentRouter you need to give the student model a type in the app router 
```
import { postRouter } from "@/server/api/routers/post";
import { createTRPCRouter } from "@/server/api/trpc";
import { studentRouter } from "./routers/student";
/**
 * This is the primary router for your server.
 * All routers added in /api/routers should be manually added here.
 */
export const appRouter = createTRPCRouter({
  post: postRouter,
  student : studentRouter,  // this is the student router we created
});
// export type definition of API

export type AppRouter = typeof appRouter;
```
**step 7** : now to display the form on the client side we need to create table.tsx to display it 
**step 8** : now in order to fetch the student detail we can fetch it 
```
const students = await api.student.findMany.query();  // if you are not mentioning the student router in the root.ts you may get the error 
```
**step 9** : map the values to display on the client side 
```
 <TableBody className="">

          {students?.map((student) => (
            <TableRow key={student.id}>
              <TableCell>{student.name}</TableCell>
              <TableCell>{student.department}</TableCell>
              <TableCell>{student.gender}</TableCell>
              <TableCell>{student.category}</TableCell>
              <TableCell>{student.batch}</TableCell>
              <TableCell>{student.address}</TableCell>
            </TableRow>
          ))}
        </TableBody>
```
**step 10** : now in the forms we need to write the logic in onSubmit function so we need to create mutation in the studentRouter 
```
 createStudent: publicProcedure
    .input(
      z.object({
        name: z.string(),
        department: z.string(),
        gender: z.string(),
        category: z.string(),
        batch: z.number(),
        address: z.string(),
      }),
    )
    .mutation(async ({ ctx, input }) => {
      const students = await ctx.db.student.create({
        data: {
          name: input.name,
          department: input.department,
          gender: input.gender,
          category: input.category,
          batch: input.batch,
          address: input.address,
        },
      });
      return students;
    }),
```
**step 11** now on the form page write the logic for onSubmit
```
const createStudents = api.student.createStudent.useMutation({
    onSuccess: () => {
      router.refresh();
    },
  });
  function onSubmit(values: z.infer<typeof formSchema>) {
    startTransition(async () => {
      await createStudents.mutateAsync({
        name: values.name,
        department: values.department,
        gender: values.gender,
        category: values.category,
        batch: parseInt(values.batch),
        address: values.address,
      });
      router.push("/detail");
    });
  }
```
**step 12** : follow step 2,3,4 to create an update form (seperate file eg: EditFrom.tsx)
**step 13** : in( src/server/api/routers/student.ts ) student router we need to create update mutation  (similar logic in step 10)
```
editStudent: publicProcedure

    .input(
      z.object({
        id: z.number(),
        name: z.string(),
        department: z.string(),
        gender: z.string(),
        category: z.string(),
        batch: z.number(),
        address: z.string(),
      }),
    )

    .mutation(async ({ ctx, input }) => {
      const updateStudents = await ctx.db.student.update({
        where: {
          id: input.id,
        },
        data: {
          name: input.name,
          department: input.department,
          gender: input.gender,
          category: input.category,
          batch: input.batch,
          address: input.address,
        },
      });
      return updateStudents;
    }),
```
**step 14** : to prefil the default values in the update form 
for example if your going to update the student detail of id : 2 (name : ben , age : 20) , in the update form the existing values of that student id : 2 (default value of name : ben , age : 20) will be there and we can customize or change it 
 **Step 15** : in order to do fetch the default values to the update form we need to write an query procedure (in student router)

```
//prefill form with student data to edit

  getStudent: publicProcedure
    .input(
      z.object({
        id: z.number(),
      }),
    )
    .query(async ({ ctx, input }) => {
      const student = await ctx.db.student.findUnique({
        where: {
          id: input.id,
        },
      });
      return student;
    }),
```
**step 16** :  getting the default values in the update form we need to use an useEffect to set values (in the editForm.tsx)

```
 useEffect(() => {

    if (student) {
      form.setValue("name", student.name);
      form.setValue("department", student.department);
      form.setValue("gender", student.gender);
      form.setValue("category", student.category);
      form.setValue("batch", String(student.batch));
      form.setValue("address", student.address);
    }
  }, [student, form]);
  console.log(student);
```
**step 17** : onsubmit function in update form 

```
 const editStudents = api.student.editStudent.useMutation({
    onSuccess: () => {
      toast.success("Student updated successfully");
      router.push("/detail");
    },
    onError: (error) => {
      toast.error(error.message);
    },
  });

  

  function onSubmit(values: z.infer<typeof updateFormSchema>) {
    if (student) {
      startTransition(async () => {
        await editStudents.mutateAsync({
          id: student.id,
          name: values.name,
          department: values.department,
          gender: values.gender,
          category: values.category,
          batch: Number(values.batch),
          address: values.address,
        });
        toast.success("Student detail successfully added!");
        router.push("/detail");
      });
    }
  }
```