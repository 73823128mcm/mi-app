import tkinter as tk
from tkinter import messagebox, ttk
from datetime import datetime
import json

class Mascota:
    def __init__(self, nombre, tipo, edad, peso, necesidades, fecha_registro=None):
        self.nombre = nombre
        self.tipo = tipo
        self.edad = edad
        self.peso = peso
        self.necesidades = necesidades
        self.fecha_registro = fecha_registro or datetime.now().strftime("%Y-%m-%d %H:%M")
        self.historial_cuidados = []

    def agregar_cuidado(self, cuidado):
        fecha = datetime.now().strftime("%Y-%m-%d %H:%M")
        self.historial_cuidados.append({"fecha": fecha, "descripcion": cuidado})

    def __str__(self):
        return f"{self.nombre} - {self.tipo} ({self.edad} años) - {self.peso}kg"

    def to_dict(self):
        return {
            "nombre": self.nombre,
            "tipo": self.tipo,
            "edad": self.edad,
            "peso": self.peso,
            "necesidades": self.necesidades,
            "fecha_registro": self.fecha_registro,
            "historial_cuidados": self.historial_cuidados
        }

    @staticmethod
    def from_dict(data):
        mascota = Mascota(
            data["nombre"],
            data["tipo"],
            data["edad"],
            data["peso"],
            data["necesidades"],
            data.get("fecha_registro")
        )
        mascota.historial_cuidados = data.get("historial_cuidados", [])
        return mascota


class AppCuidadoAnimal:
    def __init__(self, ventana):
        self.ventana = ventana
        self.ventana.title(" App de Cuidado Animal")
        self.ventana.geometry("900x650")
        self.ventana.configure(bg="#f0f4f8")
        
        self.mascotas = []
        self.mascota_seleccionada = None
        
        self.crear_interfaz()
        self.cargar_datos()

    def crear_interfaz(self):
        # Frame principal con dos columnas
        main_frame = tk.Frame(self.ventana, bg="#f0f4f8")
        main_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

        # ============ COLUMNA IZQUIERDA: Formulario ============
        left_frame = tk.Frame(main_frame, bg="white", relief=tk.RAISED, bd=2)
        left_frame.grid(row=0, column=0, sticky="nsew", padx=(0, 10))
        
        # Título
        titulo = tk.Label(
            left_frame,
            text=" Registrar Mascota",
            font=("Helvetica", 16, "bold"),
            bg="white",
            fg="#2c3e50"
        )
        titulo.pack(pady=15)

        # Frame para campos del formulario
        form_frame = tk.Frame(left_frame, bg="white")
        form_frame.pack(padx=20, pady=10, fill=tk.BOTH)

        # Campo Nombre
        tk.Label(form_frame, text=" Nombre:", font=("Helvetica", 10), bg="white", anchor="w").grid(
            row=0, column=0, sticky="w", pady=5
        )
        self.entry_nombre = tk.Entry(form_frame, font=("Helvetica", 10), width=25)
        self.entry_nombre.grid(row=0, column=1, pady=5, padx=5)

        # Campo Tipo
        tk.Label(form_frame, text=" Tipo:", font=("Helvetica", 10), bg="white", anchor="w").grid(
            row=1, column=0, sticky="w", pady=5
        )
        self.combo_tipo = ttk.Combobox(
            form_frame,
            values=["Perro", "Gato", "Conejo", "Hámster", "Pájaro", "Reptil", "Otro"],
            font=("Helvetica", 10),
            width=23,
            state="readonly"
        )
        self.combo_tipo.grid(row=1, column=1, pady=5, padx=5)
        self.combo_tipo.set("Perro")

        # Campo Edad
        tk.Label(form_frame, text=" Edad (años):", font=("Helvetica", 10), bg="white", anchor="w").grid(
            row=2, column=0, sticky="w", pady=5
        )
        self.entry_edad = tk.Entry(form_frame, font=("Helvetica", 10), width=25)
        self.entry_edad.grid(row=2, column=1, pady=5, padx=5)

        # Campo Peso
        tk.Label(form_frame, text=" Peso (kg):", font=("Helvetica", 10), bg="white", anchor="w").grid(
            row=3, column=0, sticky="w", pady=5
        )
        self.entry_peso = tk.Entry(form_frame, font=("Helvetica", 10), width=25)
        self.entry_peso.grid(row=3, column=1, pady=5, padx=5)

        # Campo Necesidades
        tk.Label(form_frame, text=" Necesidades:", font=("Helvetica", 10), bg="white", anchor="w").grid(
            row=4, column=0, sticky="w", pady=5
        )
        self.text_necesidades = tk.Text(form_frame, font=("Helvetica", 10), width=25, height=4)
        self.text_necesidades.grid(row=4, column=1, pady=5, padx=5)

        # Botones de acción
        btn_frame = tk.Frame(left_frame, bg="white")
        btn_frame.pack(pady=15)

        self.btn_agregar = tk.Button(
            btn_frame,
            text=" Agregar Mascota",
            command=self.agregar_mascota,
            bg="#27ae60",
            fg="white",
            font=("Helvetica", 11, "bold"),
            relief=tk.FLAT,
            padx=20,
            pady=8,
            cursor="hand2"
        )
        self.btn_agregar.pack(side=tk.LEFT, padx=5)

        self.btn_actualizar = tk.Button(
            btn_frame,
            text="✏️ Actualizar",
            command=self.actualizar_mascota,
            bg="#3498db",
            fg="white",
            font=("Helvetica", 11, "bold"),
            relief=tk.FLAT,
            padx=20,
            pady=8,
            cursor="hand2",
            state=tk.DISABLED
        )
        self.btn_actualizar.pack(side=tk.LEFT, padx=5)

        btn_limpiar = tk.Button(
            btn_frame,
            text="🗑️ Limpiar",
            command=self.limpiar_campos,
            bg="#95a5a6",
            fg="white",
            font=("Helvetica", 11, "bold"),
            relief=tk.FLAT,
            padx=20,
            pady=8,
            cursor="hand2"
        )
        btn_limpiar.pack(side=tk.LEFT, padx=5)

        # ============ COLUMNA DERECHA: Lista y detalles ============
        right_frame = tk.Frame(main_frame, bg="white", relief=tk.RAISED, bd=2)
        right_frame.grid(row=0, column=1, sticky="nsew")
        
        # Configurar peso de columnas
        main_frame.columnconfigure(0, weight=1)
        main_frame.columnconfigure(1, weight=2)
        main_frame.rowconfigure(0, weight=1)

        # Título lista
        titulo_lista = tk.Label(
            right_frame,
            text=" Mascotas Registradas",
            font=("Helvetica", 16, "bold"),
            bg="white",
            fg="#2c3e50"
        )
        titulo_lista.pack(pady=15)

        # Búsqueda
        search_frame = tk.Frame(right_frame, bg="white")
        search_frame.pack(padx=20, pady=5)
        
        tk.Label(search_frame, text="BUSCAR", font=("Helvetica", 12), bg="white").pack(side=tk.LEFT)
        self.entry_buscar = tk.Entry(search_frame, font=("Helvetica", 10), width=30)
        self.entry_buscar.pack(side=tk.LEFT, padx=5)
        self.entry_buscar.bind("<KeyRelease>", self.buscar_mascota)

        # Lista de mascotas con scrollbar
        list_frame = tk.Frame(right_frame, bg="white")
        list_frame.pack(padx=20, pady=10, fill=tk.BOTH, expand=True)

        scrollbar = tk.Scrollbar(list_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.lista_mascotas = tk.Listbox(
            list_frame,
            font=("Helvetica", 10),
            yscrollcommand=scrollbar.set,
            selectmode=tk.SINGLE,
            relief=tk.FLAT,
            bg="#ecf0f1"
        )
        self.lista_mascotas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.config(command=self.lista_mascotas.yview)
        self.lista_mascotas.bind("<<ListboxSelect>>", self.seleccionar_mascota)

        # Botones de gestión
        btn_gestion_frame = tk.Frame(right_frame, bg="white")
        btn_gestion_frame.pack(pady=10)

        tk.Button(
            btn_gestion_frame,
            text=" Ver Detalles",
            command=self.ver_detalles,
            bg="#3498db",
            fg="white",
            font=("Helvetica", 10, "bold"),
            relief=tk.FLAT,
            padx=15,
            pady=5,
            cursor="hand2"
        ).pack(side=tk.LEFT, padx=5)

        tk.Button(
            btn_gestion_frame,
            text=" Agregar Cuidado",
            command=self.agregar_cuidado,
            bg="#9b59b6",
            fg="white",
            font=("Helvetica", 10, "bold"),
            relief=tk.FLAT,
            padx=15,
            pady=5,
            cursor="hand2"
        ).pack(side=tk.LEFT, padx=5)

        tk.Button(
            btn_gestion_frame,
            text=" Eliminar",
            command=self.eliminar_mascota,
            bg="#e74c3c",
            fg="white",
            font=("Helvetica", 10, "bold"),
            relief=tk.FLAT,
            padx=15,
            pady=5,
            cursor="hand2"
        ).pack(side=tk.LEFT, padx=5)

        # Label contador
        self.label_contador = tk.Label(
            right_frame,
            text="Total: 0 mascotas",
            font=("Helvetica", 10),
            bg="white",
            fg="#7f8c8d"
        )
        self.label_contador.pack(pady=5)

    def agregar_mascota(self):
        nombre = self.entry_nombre.get().strip()
        tipo = self.combo_tipo.get()
        edad = self.entry_edad.get().strip()
        peso = self.entry_peso.get().strip()
        necesidades = self.text_necesidades.get("1.0", tk.END).strip()

        if not nombre or not edad or not peso:
            messagebox.showwarning(" Campos incompletos", "Por favor completa los campos: Nombre, Edad y Peso.")
            return

        try:
            edad = int(edad)
            peso = float(peso)
            if edad < 0 or peso < 0:
                raise ValueError
        except ValueError:
            messagebox.showerror(" Error", "Edad y Peso deben ser números positivos.")
            return

        mascota = Mascota(nombre, tipo, edad, peso, necesidades)
        self.mascotas.append(mascota)
        self.actualizar_lista()
        self.limpiar_campos()
        self.guardar_datos()
        messagebox.showinfo(" Éxito", f"¡{nombre} ha sido registrado(a) exitosamente!")

    def actualizar_mascota(self):
        if self.mascota_seleccionada is None:
            messagebox.showwarning(" Advertencia", "Por favor selecciona una mascota para actualizar.")
            return

        nombre = self.entry_nombre.get().strip()
        tipo = self.combo_tipo.get()
        edad = self.entry_edad.get().strip()
        peso = self.entry_peso.get().strip()
        necesidades = self.text_necesidades.get("1.0", tk.END).strip()

        if not nombre or not edad or not peso:
            messagebox.showwarning(" Campos incompletos", "Por favor completa todos los campos obligatorios.")
            return

        try:
            edad = int(edad)
            peso = float(peso)
            if edad < 0 or peso < 0:
                raise ValueError
        except ValueError:
            messagebox.showerror(" Error", "Edad y Peso deben ser números positivos.")
            return

        mascota = self.mascotas[self.mascota_seleccionada]
        mascota.nombre = nombre
        mascota.tipo = tipo
        mascota.edad = edad
        mascota.peso = peso
        mascota.necesidades = necesidades

        self.actualizar_lista()
        self.limpiar_campos()
        self.guardar_datos()
        messagebox.showinfo(" Actualizado", f"¡{nombre} ha sido actualizado(a) exitosamente!")

    def seleccionar_mascota(self, event):
        seleccion = self.lista_mascotas.curselection()
        if not seleccion:
            return

        index = seleccion[0]
        self.mascota_seleccionada = index
        mascota = self.mascotas[index]

        self.entry_nombre.delete(0, tk.END)
        self.entry_nombre.insert(0, mascota.nombre)
        
        self.combo_tipo.set(mascota.tipo)
        
        self.entry_edad.delete(0, tk.END)
        self.entry_edad.insert(0, mascota.edad)
        
        self.entry_peso.delete(0, tk.END)
        self.entry_peso.insert(0, mascota.peso)
        
        self.text_necesidades.delete("1.0", tk.END)
        self.text_necesidades.insert("1.0", mascota.necesidades)

        self.btn_actualizar.config(state=tk.NORMAL)
        self.btn_agregar.config(state=tk.DISABLED)

    def ver_detalles(self):
        seleccion = self.lista_mascotas.curselection()
        if not seleccion:
            messagebox.showinfo("ℹ Información", "Por favor selecciona una mascota.")
            return

        index = seleccion[0]
        m = self.mascotas[index]

        detalles = f"""
 INFORMACIÓN DE LA MASCOTA

 Nombre: {m.nombre}
 Tipo: {m.tipo}
 Edad: {m.edad} años
 Peso: {m.peso} kg
 Registrado: {m.fecha_registro}

 Necesidades:
{m.necesidades if m.necesidades else 'Ninguna especificada'}

 Historial de Cuidados ({len(m.historial_cuidados)}):
"""
        if m.historial_cuidados:
            for cuidado in m.historial_cuidados[-5:]:  # Últimos 5
                detalles += f"\n• {cuidado['fecha']}: {cuidado['descripcion']}"
        else:
            detalles += "\nSin registros aún"

        messagebox.showinfo(" Detalles de la Mascota", detalles)

    def agregar_cuidado(self):
        seleccion = self.lista_mascotas.curselection()
        if not seleccion:
            messagebox.showinfo("ℹ Información", "Por favor selecciona una mascota.")
            return

        # Ventana para agregar cuidado
        ventana_cuidado = tk.Toplevel(self.ventana)
        ventana_cuidado.title("Agregar Cuidado")
        ventana_cuidado.geometry("400x200")
        ventana_cuidado.configure(bg="white")
        ventana_cuidado.transient(self.ventana)
        ventana_cuidado.grab_set()

        tk.Label(
            ventana_cuidado,
            text=" Descripción del Cuidado:",
            font=("Helvetica", 11, "bold"),
            bg="white"
        ).pack(pady=10)

        text_cuidado = tk.Text(ventana_cuidado, font=("Helvetica", 10), width=40, height=5)
        text_cuidado.pack(pady=10, padx=20)

        def guardar_cuidado():
            descripcion = text_cuidado.get("1.0", tk.END).strip()
            if not descripcion:
                messagebox.showwarning(" Advertencia", "Por favor escribe una descripción.")
                return
            
            index = seleccion[0]
            self.mascotas[index].agregar_cuidado(descripcion)
            self.guardar_datos()
            messagebox.showinfo(" Éxito", "¡Cuidado registrado exitosamente!")
            ventana_cuidado.destroy()

        tk.Button(
            ventana_cuidado,
            text=" Guardar",
            command=guardar_cuidado,
            bg="#27ae60",
            fg="white",
            font=("Helvetica", 10, "bold"),
            relief=tk.FLAT,
            padx=20,
            pady=5,
            cursor="hand2"
        ).pack(pady=10)

    def eliminar_mascota(self):
        seleccion = self.lista_mascotas.curselection()
        if not seleccion:
            messagebox.showinfo("ℹ Información", "Por favor selecciona una mascota.")
            return

        index = seleccion[0]
        nombre = self.mascotas[index].nombre

        confirmar = messagebox.askyesno(
            "⚠️ Confirmar Eliminación",
            f"¿Estás seguro de eliminar a {nombre}?\nEsta acción no se puede deshacer."
        )

        if confirmar:
            del self.mascotas[index]
            self.actualizar_lista()
            self.limpiar_campos()
            self.guardar_datos()
            messagebox.showinfo(" Eliminado", f"{nombre} ha sido eliminado(a) del sistema.")

    def buscar_mascota(self, event):
        termino = self.entry_buscar.get().lower()
        self.lista_mascotas.delete(0, tk.END)

        for mascota in self.mascotas:
            if termino in mascota.nombre.lower() or termino in mascota.tipo.lower():
                self.lista_mascotas.insert(tk.END, str(mascota))

        if not termino:
            self.actualizar_lista()

    def actualizar_lista(self):
        self.lista_mascotas.delete(0, tk.END)
        for mascota in self.mascotas:
            self.lista_mascotas.insert(tk.END, str(mascota))
        
        self.label_contador.config(text=f"Total: {len(self.mascotas)} mascota(s)")

    def limpiar_campos(self):
        self.entry_nombre.delete(0, tk.END)
        self.combo_tipo.set("Perro")
        self.entry_edad.delete(0, tk.END)
        self.entry_peso.delete(0, tk.END)
        self.text_necesidades.delete("1.0", tk.END)
        self.mascota_seleccionada = None
        self.btn_actualizar.config(state=tk.DISABLED)
        self.btn_agregar.config(state=tk.NORMAL)

    def guardar_datos(self):
        try:
            with open("mascotas_data.json", "w", encoding="utf-8") as f:
                datos = [m.to_dict() for m in self.mascotas]
                json.dump(datos, f, indent=2, ensure_ascii=False)
        except Exception as e:
            print(f"Error al guardar: {e}")

    def cargar_datos(self):
        try:
            with open("mascotas_data.json", "r", encoding="utf-8") as f:
                datos = json.load(f)
                self.mascotas = [Mascota.from_dict(d) for d in datos]
                self.actualizar_lista()
        except FileNotFoundError:
            pass
        except Exception as e:
            print(f"Error al cargar: {e}")


# Crear y ejecutar la aplicación
if __name__ == "__main__":
    ventana = tk.Tk()
    app = AppCuidadoAnimal(ventana)
    ventana.mainloop()
