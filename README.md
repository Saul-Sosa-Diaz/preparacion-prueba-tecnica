# preparacion-prueba-tecnica

import logging
from pathlib import Path
import pandas as pd

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
logger = logging.getLogger(__name__)

# Configuración y Constantes
CONFIG = {
    "files": {
        "input_2023": Path("Datos_2023.xlsx"),
        "input_2024": Path("Datos_2024.xlsx"),
        "output": Path("resultado.xlsx"),
    },
    "columns": {
        "cancelation_date": "Datfechaanulacion",
        "rejection_date": "Datfecharechazo",
        "organica": "Strorganica",
        "ejercicio": "Strejercicio",
        "importe": "Importe Imp",
    },
    "filters": {
        "organica_exclude_substring": "AA",
    },
    "dtypes": {
        "Strorganica": "string",
        "Strejercicio": "string",
    }
}


def load_dataset(file_path: Path, dtypes: dict | None = None) -> pd.DataFrame:
    """Carga un archivo Excel validando su existencia y aplicando dtypes explícitos."""
    if not file_path.exists():
        raise FileNotFoundError(f"Archivo no encontrado: {file_path}")
    
    logger.info(f"Cargando dataset desde {file_path}...")
    return pd.read_excel(file_path, engine="openpyxl", dtype=dtypes)


def validate_schema(df: pd.DataFrame, required_columns: list[str]) -> None:
    """Valida que el DataFrame contenga todas las columnas requeridas."""
    missing_cols = [col for col in required_columns if col not in df.columns]
    if missing_cols:
        raise ValueError(f"Faltan columnas requeridas en el DataFrame: {missing_cols}")


def clean_and_aggregate_data(df: pd.DataFrame, cols: dict, exclude_sub: str) -> pd.DataFrame:
    """Aplica el filtrado de nulos, agregación y exclusión de subcadenas."""
    # 1. Filtrar registros activos (sin fecha de anulación ni rechazo)
    active_mask = df[cols["cancelation_date"]].isna() & df[cols["rejection_date"]].isna()
    active_df = df.loc[active_mask]
    
    # 2. Agrupación por orgánica y ejercicio
    agg_df = (
        active_df
        .groupby([cols["organica"], cols["ejercicio"]], as_index=False)[cols["importe"]]
        .sum()
    )
    
    # 3. Filtrar subcadena y ordenar
    valid_organica_mask = ~agg_df[cols["organica"]].str.contains(exclude_sub, na=False)
    
    result_df = (
        agg_df
        .loc[valid_organica_mask]
        .sort_values(by=[cols["organica"], cols["ejercicio"]])
    )
    
    return result_df


def export_to_excel(df: pd.DataFrame, output_path: Path) -> None:
    """Exporta el DataFrame procesado a un archivo Excel."""
    output_path.parent.mkdir(parents=True, exist_ok=True)
    df.to_excel(output_path, index=False, engine="openpyxl")
    logger.info(f"Dataset exportado exitosamente a: {output_path} ({len(df)} filas)")


def main() -> None:
    cols = CONFIG["columns"]
    dtypes = CONFIG["dtypes"]
    files = CONFIG["files"]
    
    # 1. Carga y concatenación
    df_2023 = load_dataset(files["input_2023"], dtypes=dtypes)
    df_2024 = load_dataset(files["input_2024"], dtypes=dtypes)
    raw_df = pd.concat([df_2023, df_2024], ignore_index=True)
    
    # 2. Validación
    validate_schema(raw_df, list(cols.values()))
    logger.info(f"Filas totales cargadas: {raw_df.shape[0]}")
    
    # 3. Transformación
    processed_df = clean_and_aggregate_data(
        df=raw_df,
        cols=cols,
        exclude_sub=CONFIG["filters"]["organica_exclude_substring"]
    )
    
    # 4. Exportación
    export_to_excel(processed_df, files["output"])


if __name__ == "__main__":
    try:
        main()
    except Exception as err:
        logger.critical(f"Fallo crítico en la ejecución del pipeline: {err}", exc_info=True)
