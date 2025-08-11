"""
Calculadora_documentada.py

Descripción:
    Versión documentada del script de una calculadora hecha con Tkinter.
    El archivo contiene el código original comentado detalladamente, una explicación
    de cada parte (clases, métodos y configuración de la interfaz) y una
    implementación opcional más segura para evaluar expresiones aritméticas.

Requisitos:
    - Python 3.8+ (Tkinter viene incluido en la mayoría de las distribuciones de CPython)

Cómo ejecutar:
    python Calculadora_documentada.py

Resumen de contenidos:
    - Clase Calculadora: construye la interfaz y maneja eventos de botones.
    - crear_botones(): coloca los botones en una cuadrícula y los configura.
    - click_boton(valor): lógica para manejar cada botón (números, operadores, borrar, etc.).
    - safe_eval(expr): (opcional) función segura para evaluar expresiones aritméticas
      evitando el uso directo de eval(). Se recomienda usar esta función en vez de eval
      si la entrada puede provenir de usuarios no confiables.

Mejoras sugeridas (más abajo se muestran ejemplos/explicaciones):
    - Reemplazar eval() por una evaluación segura (se incluye ejemplo con ast).
    - Agregar atajos de teclado (teclas 0-9, +, -, *, /, Enter, Backspace, Esc).
    - Soporte para paréntesis, exponentes y validación de entrada más robusta.
    - Internacionalización (por ejemplo, usar coma "," como separador decimal si se desea).

"""

import tkinter as tk
from tkinter import messagebox
import ast
import operator as _op

# -----------------------------
# Evaluador seguro (opcional)
# -----------------------------
# Motivo: el uso de eval() ejecuta cualquier expresión de Python, lo cual es peligroso
# si el contenido proviene de usuarios no confiables. A continuación se incluye una
# función basada en ast que permite únicamente operaciones aritméticas.

# Mapeo de nodos AST a funciones de operador
_OPERADORES = {
    ast.Add: _op.add,
    ast.Sub: _op.sub,
    ast.Mult: _op.mul,
    ast.Div: _op.truediv,
    ast.Pow: _op.pow,
    ast.Mod: _op.mod,
    ast.USub: _op.neg,
    ast.UAdd: _op.pos,
}


def _eval_ast(node):
    """Evalúa recursivamente un nodo AST permitiendo sólo operaciones seguras.

    Permite: números (int/float), operadores binarios (+ - * / % **), operadores
    unarios (+ -) y paréntesis implícitos por la estructura AST.
    """
    # Binaria: left <op> right
    if isinstance(node, ast.BinOp):
        left = _eval_ast(node.left)
        right = _eval_ast(node.right)
        op_type = type(node.op)
        if op_type in _OPERADORES:
            return _OPERADORES[op_type](left, right)
        raise ValueError(f"Operador no permitido: {op_type}")

    # Unario: +x or -x
    if isinstance(node, ast.UnaryOp):
        operand = _eval_ast(node.operand)
        op_type = type(node.op)
        if op_type in _OPERATORS:
            return _OPERATORS[op_type](operand)
        raise ValueError(f"Operador unario no permitido: {op_type}")

    # Números: ast.Constant (Python 3.8+) o ast.Num (anteriores)
    if isinstance(node, ast.Constant):
        if isinstance(node.value, (int, float)):
            return node.value
        raise ValueError("Sólo se permiten valores numéricos")
    if isinstance(node, ast.Num):
        return node.n

    # No permitimos nombres, llamadas, atributos, etc.
    raise ValueError(f"Nodo no permitido: {type(node)}")


def safe_eval(expr: str):
    """Evalúa una expresión aritmética de forma segura.

    Args:
        expr (str): expresión aritmética, p. ej. "2+3*(4-1)/2"

    Returns:
        int | float: resultado numérico de la expresión.

    Raises:
        ValueError: si la expresión contiene nodos o tokens no permitidos.
    """
    try:
        parsed = ast.parse(expr, mode='eval')
        return _eval_ast(parsed.body)
    except Exception as e:
        # Normalizamos la excepción para el llamador
        raise ValueError("Expresión no válida o no permitida") from e


# -----------------------------
# Clase principal: Calculadora
# -----------------------------
class Calculadora:
    """Interfaz gráfica de una calculadora básica usando Tkinter.

    Atributos principales:
        root (tk.Tk): ventana principal de la aplicación.
        entrada (tk.Entry): widget donde se muestra y edita la expresión.

    Métodos principales:
        crear_botones(): genera y organiza los botones en la interfaz.
        click_boton(valor): maneja la interacción cuando se pulsa un botón.

    Nota de diseño:
        - El layout usa un Frame interno (frame_botones) para organizar la cuadrícula.
        - Los botones usan columnspan para ocupar más de una columna (por ejemplo '0').
        - El diseño actual fija el tamaño de la ventana (geometry + resizable(False))
          por simplicidad; si se desea una interfaz responsive, eliminar esa restricción
          y ajustar los pesos de filas/columnas.
    """

    def __init__(self, root: tk.Tk):
        """Inicializa la ventana principal y los widgets.

        Args:
            root (tk.Tk): instancia de Tk que representa la ventana principal.
        """
        self.root = root
        self.root.title("Calculadora")

        # Estética: colores, tamaño y comportamiento de la ventana
        self.root.configure(bg="#2b2b2b")
        self.root.geometry("375x550")  # ancho x alto
        self.root.resizable(False, False)

        # Entry para mostrar la entrada/resultado. justify='right' para apariencia
        self.entrada = tk.Entry(
            root,
            width=17,
            font=("arial", 28),
            borderwidth=0,
            relief="solid",
            bg="#2b2b2b",
            fg="white",
            justify="right",
        )
        # grid coloca el entry en la fila 0 y ocupa las 4 columnas virtuales
        self.entrada.grid(row=0, column=0, columnspan=4, ipadx=8, ipady=10, pady=(10, 5))

        # Genera los botones
        self.crear_botones()

    def crear_botones(self):
        """Crea y dispone los botones de la calculadora.

        La lista 'botones' contiene tuplas (texto, columnspan) para indicar la
        etiqueta del botón y cuántas columnas ocupa en la cuadrícula. El diccionario
        'colores_botones' centraliza los colores usados para facilitar cambios.
        """
        # Diseño de botones: (texto, cuanto ocupa horizontalmente)
        botones = [
            ("C", 2), ("←", 1), ("/", 1),
            ("7", 1), ("8", 1), ("9", 1), ("*", 1),
            ("4", 1), ("5", 1), ("6", 1), ("-", 1),
            ("1", 1), ("2", 1), ("3", 1), ("+", 1),
            ("0", 2), (".", 1), ("=", 1)
        ]

        # Colores y estilos reutilizables
        colores_botones = {
            "numero": "#4d4d4d",
            "operador": "#fe9505",
            "igual": "#fe9505",
            "fondo": "#2b2b2b",
            "texto": "#ffffff",
            "reset": "#d32f2f",
            "borrar": "#fe9505",
        }

        frame_botones = tk.Frame(self.root, bg=colores_botones["fondo"])
        frame_botones.grid(row=1, column=0, columnspan=4, pady=(0, 10))

        fila = 0
        columna = 0

        # Iteramos sobre la lista y creamos cada botón.
        for boton, span in botones:
            # Selección de color según tipo de botón
            if boton in ["/", "*", "-", "+", "=", "←"]:
                color_fondo = colores_botones["operador"]
            else:
                color_fondo = colores_botones["numero"]

            if boton == "C":
                color_fondo = colores_botones["reset"]
            elif boton == "←":
                color_fondo = colores_botones["borrar"]
            elif boton == "=":
                color_fondo = colores_botones["igual"]

            # Observación: se usa lambda b=boton: ... para capturar correctamente el
            # valor del botón dentro del bucle (evita la referencia tardía a 'boton').
            btn = tk.Button(
                frame_botones,
                text=boton,
                width=5 * span,    # ancho fijo por simplicidad; se puede eliminar para que
                                   # la cuadrícula distribuya el espacio automáticamente.
                height=2,
                font=("arial", 20),
                bg=color_fondo,
                fg=colores_botones["texto"],
                borderwidth=0,
                command=lambda b=boton: self.click_boton(b),
            )

            # Colocamos el botón en la cuadrícula, usando columnspan cuando corresponde.
            btn.grid(row=fila, column=columna, columnspan=span, padx=1, pady=1, sticky="nsew")

            columna += span
            # Cuando llegamos a 4 columnas virtuales pasamos a la siguiente fila
            if columna >= 4:
                columna = 0
                fila += 1

        # Configuramos pesos para que la cuadrícula del frame escale si la ventana cambia.
        # Actualmente la ventana está fijada (resizable(False, False)), pero esto es útil si
        # se desea permitir redimensionado en el futuro.
        for i in range(4):
            frame_botones.grid_columnconfigure(i, weight=1)
        for i in range(fila + 1):
            frame_botones.grid_rowconfigure(i, weight=1)

    def click_boton(self, valor: str):
        """Manejador central para todas las acciones de los botones.

        Parámetros:
            valor (str): texto del botón pulsado (por ejemplo '1', '+', 'C', '=', '←').
        """
        # Botón '=' -> evaluar la expresión actual
        if valor == "=":
            try:
                expr = self.entrada.get()

                # Opción A (original): usar eval (rápido pero inseguro si la entrada no es
                # de confianza)
                # resultado = str(eval(expr))

                # Opción B (recomendada): usar safe_eval() definido arriba.
                # Evita ejecución arbitraria de código.
                resultado = str(safe_eval(expr))

                # Actualizamos el campo de entrada con el resultado
                self.entrada.delete(0, tk.END)
                self.entrada.insert(0, resultado)
            except Exception:
                # Mostramos un cuadro de error y limpiamos la entrada para que el usuario reintente
                messagebox.showerror("Error", "Entrada no válida")
                self.entrada.delete(0, tk.END)

        # Botón 'C' -> limpiar toda la entrada
        elif valor == "C":
            self.entrada.delete(0, tk.END)

        # Botón '←' -> borrar el último carácter
        elif valor == "←":
            current = self.entrada.get()
            # Borramos el último carácter si existe
            if current:
                self.entrada.delete(len(current) - 1, tk.END)

        # Cualquier otro botón (números, operadores, punto decimal) -> insertar texto
        else:
            self.entrada.insert(tk.END, valor)


# -----------------------------
# Ejecutable directo
# -----------------------------
if __name__ == "__main__":
    root = tk.Tk()
    app = Calculadora(root)
    root.mainloop()

# -----------------------------
# Notas finales y recomendaciones
# -----------------------------
# 1) Seguridad: usar safe_eval() cuando la entrada pueda venir de usuarios externos.
# 2) Mejoras de usabilidad: añadir atajos de teclado, soporte para copiar/pegar,
#    y manejo de coma como separador decimal si se desea compatibilidad local.
# 3) Internacionalización: si tu público usa coma "," para decimales, considera
#    reemplazarlas por puntos antes de evaluar y mostrar resultados con la convención apropiada.
# 4) Paquetes/instalación: si quieres distribuir esto como ejecutable, pyinstaller
#    o similar puede crear un .exe a partir del script.

# Si quieres, puedo:
# - Aplicar estos cambios directamente a tu archivo original.
# - Añadir atajos de teclado (ej.: Enter = '=', Backspace = '←', Esc = 'C').
# - Convertir los botones a estilos ttk para apariencia nativa.
# - Crear un README más corto con instrucciones de uso para otros colaboradores.
