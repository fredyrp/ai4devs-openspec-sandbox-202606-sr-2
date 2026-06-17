# Entrega

## Evidencia de que OpenSpec está operativo — pega el output de openspec --version y de ls -R openspec/.

```bash
ai4devs-openspec-sandbox-202606-sr-2 > openspec --version
1.4.1
```

```bash
ai4devs-openspec-sandbox-202606-sr-2 > ls -R .\openspec\

Directory: C:\XAVTEF\growth\LIDR\modulo2_sdd\ai4devs-openspec-sandbox-202606-sr-2\openspec

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r--     16/06/2026  11:18 p. m.                  changes
d-r--     16/06/2026  11:18 p. m.                  specs
-a---     16/06/2026  11:18 p. m.            573 󰉢  config.yaml

Directory: C:\XAVTEF\growth\LIDR\modulo2_sdd\ai4devs-openspec-sandbox-202606-sr-2\openspec\changes

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r--     16/06/2026  11:18 p. m.                  archive
```

## Plantilla de los 3 Pilares rellenada con tu micro-tarea.

* **Micro-tarea:** 
  
  Reconocedor de instruccones en CSV.

* **Pilar 1 — Herramienta:**
  
    Github Copilot en VSCode.

    Normalmente trabajo con VSCode, dispogo de una licencia para el usao de Copilot y se ma hece muy sencillo usarlo.

* **Pilar 2 — Contexto:** 
  
    Lenaguaje: python

    Las instrucciones son lineas en formato CSV: 
    
    Donde:
    
    - Posición 1: Consecutivo númerico que empieza en 1 asta n, este consecutivo especifica el orden enque deben ejecutarse las instricciones.
    - Posición 2: Especifica el nombre o el tipo de instrccion a ejecutar.
    - Posisicon 3 - n: Especifica los parametros correspondientes a cada instruccion.
  
    Ejemplos:
    
    ```
    1,suma,$resp,1,1
    2,concatenar,$cadena,"Respuesta: ",$resp
    3,imprimir,$resp
    ```
    
    Este ejemplo se identificasn 3 instrucciones suma, concatenar y imprimir, cada una con sus propios parametros.

    Los parametros con un $ al inicio especifica que se trata de una variable.

* *Pilar 3 — Prompt:* ¿Cómo lo estructuras? (estilo, formato de salida, ejemplos…)

    Creta una funcion dentro de una clase de python que reciba una cadena con una lista de instrucciones y realice las siguientes operaciones:

  - Valide que la secuencia se correta (inice de 1 a n), una instrucción por cada linea, devolver un error de secuencia en caso de no ser correcta.
  - Cree una lista de instrucciones en una estructura (clase en python) que sea capaz de alcamenar cualquier instruccion idependientemente del número de parametros que tenga.
  - Crea pruebas unitarias que validen que la funcion es capas de procesare el ejemplo y detectar cortes de secuencia.

* *Resultado:* 
  
    ¿Funcionó a la primera o tuviste que iterar?
    - Si funciono a la primera.
  
    Una mejora que harías si volvieras a hacerlo.
    - Agregar mas ejemplos al contexto y una mejor descripcion de los ejemplos.

    **instruction_parser.py**
    ```python
    from __future__ import annotations

    import csv
    from dataclasses import dataclass
    from io import StringIO


    class SequenceError(ValueError):
        """Raised when instruction sequence is not strictly 1..n."""


    @dataclass(frozen=True)
    class Instruction:
        sequence: int
        instruction_type: str
        parameters: list[str]


    class InstructionRecognizer:
        def parse_instructions(self, instructions_text: str) -> list[Instruction]:
            """Parse and validate CSV instructions from a multiline string.

            Each non-empty line must follow this format:
            sequence,instruction_type,param1,param2,...
            """
            parsed_instructions: list[Instruction] = []
            expected_sequence = 1

            for raw_line in instructions_text.splitlines():
                if not raw_line.strip():
                    continue

                row = next(csv.reader(StringIO(raw_line), skipinitialspace=True), None)
                if row is None or len(row) < 2:
                    raise ValueError(f"Invalid instruction line: {raw_line}")

                sequence_text = row[0].strip()
                if not sequence_text.isdigit():
                    raise ValueError(f"Invalid sequence value: {sequence_text}")

                sequence = int(sequence_text)
                if sequence != expected_sequence:
                    raise SequenceError(
                        f"Sequence error at line '{raw_line}': expected {expected_sequence}, got {sequence}"
                    )

                instruction_type = row[1].strip()
                if not instruction_type:
                    raise ValueError(f"Instruction type cannot be empty: {raw_line}")

                # Keep parameter payload exactly as CSV parsed it.
                parameters = row[2:]
                parsed_instructions.append(
                    Instruction(
                        sequence=sequence,
                        instruction_type=instruction_type,
                        parameters=parameters,
                    )
                )
                expected_sequence += 1

            return parsed_instructions
    ```

    **test_instruction_parser.py**
    ```python
    import unittest

    from instruction_parser import InstructionRecognizer, SequenceError


    class TestInstructionRecognizer(unittest.TestCase):
        def setUp(self) -> None:
            self.recognizer = InstructionRecognizer()

        def test_parse_valid_example(self) -> None:
            instructions_text = """1,suma,$resp,1,1
    2,concatenar,$cadena,\"Respuesta: \",$resp
    3,imprimir,$resp"""

            result = self.recognizer.parse_instructions(instructions_text)

            self.assertEqual(3, len(result))
            self.assertEqual(1, result[0].sequence)
            self.assertEqual("suma", result[0].instruction_type)
            self.assertEqual(["$resp", "1", "1"], result[0].parameters)

            self.assertEqual(2, result[1].sequence)
            self.assertEqual("concatenar", result[1].instruction_type)
            self.assertEqual(["$cadena", "Respuesta: ", "$resp"], result[1].parameters)

            self.assertEqual(3, result[2].sequence)
            self.assertEqual("imprimir", result[2].instruction_type)
            self.assertEqual(["$resp"], result[2].parameters)

        def test_sequence_gap_raises_error(self) -> None:
            instructions_text = """1,suma,$resp,1,1
    3,imprimir,$resp"""

            with self.assertRaises(SequenceError):
                self.recognizer.parse_instructions(instructions_text)

        def test_sequence_not_starting_at_one_raises_error(self) -> None:
            instructions_text = "2,suma,$resp,1,1"

            with self.assertRaises(SequenceError):
                self.recognizer.parse_instructions(instructions_text)


    if __name__ == "__main__":
        unittest.main()
    ```

## observaciones de la exploración.

- Es un framework sencillo, me parece facil de implementar.
- El flujo de trabajo que propone me parece muy generalizado creo que se podrian agregar pasos adicionales para verificaciones con test.
- Veo algunos punto para que se hagan revisiones, me parecen bien, pero se tendria que revisar en cada proceso si son suficientes.
