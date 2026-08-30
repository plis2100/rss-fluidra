import urllib.parse
import urllib.request
import xml.etree.ElementTree as ET

from datetime import datetime, timezone
from email.utils import format_datetime
from pathlib import Path

WEB_URL = "https://www.fluidra.com/es/sala-de-prensa/"
OUTPUT_FILE = Path("fluidra.xml")

CONSULTA = (
    "site:fluidra.com/es/sala-de-prensa/nota-de-prensa/ Fluidra"
)

GOOGLE_NEWS_URL = (
    "https://news.google.com/rss/search?"
    + urllib.parse.urlencode(
        {
            "q": CONSULTA,
            "hl": "es",
            "gl": "ES",
            "ceid": "ES:es",
        }
    )
)


def descargar_noticias():
    solicitud = urllib.request.Request(
        GOOGLE_NEWS_URL,
        headers={
            "User-Agent": (
                "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                "AppleWebKit/537.36 Chrome/124 Safari/537.36"
            ),
            "Accept": (
                "application/rss+xml,"
                "application/xml,text/xml"
            ),
            "Accept-Language": "es-ES,es;q=0.9",
        },
    )

    with urllib.request.urlopen(
        solicitud,
        timeout=60,
    ) as respuesta:
        contenido = respuesta.read()

    if not contenido:
        raise RuntimeError(
            "Google News ha devuelto una respuesta vacía"
        )

    raiz = ET.fromstring(contenido)
    canal = raiz.find("channel")

    if canal is None:
        raise RuntimeError(
            "Google News no ha devuelto una RSS válida"
        )

    noticias = canal.findall("item")

    if not noticias:
        raise RuntimeError(
            "No se encontraron notas de prensa de Fluidra"
        )

    return noticias


def crear_rss(noticias_originales):
    rss = ET.Element(
        "rss",
        {
            "version": "2.0",
            "xmlns:atom": "http://www.w3.org/2005/Atom",
        },
    )

    canal = ET.SubElement(rss, "channel")

    ET.SubElement(canal, "title").text = (
        "Notas de prensa de Fluidra"
    )

    ET.SubElement(canal, "link").text = WEB_URL

    ET.SubElement(canal, "description").text = (
        "Últimas notas de prensa, resultados y "
        "comunicaciones corporativas de Fluidra"
    )

    ET.SubElement(canal, "language").text = "es"

    ET.SubElement(canal, "lastBuildDate").text = (
        format_datetime(datetime.now(timezone.utc))
    )

    enlace_atom = ET.SubElement(
        canal,
        "{http://www.w3.org/2005/Atom}link",
    )

    enlace_atom.set("href", GOOGLE_NEWS_URL)
    enlace_atom.set("rel", "self")
    enlace_atom.set("type", "application/rss+xml")

    enlaces_encontrados = set()
    noticias_incluidas = 0

    for original in noticias_originales:
        titulo = original.findtext("title", "").strip()
        enlace = original.findtext("link", "").strip()
        fecha = original.findtext("pubDate", "").strip()

        descripcion = original.findtext(
            "description",
            "",
        ).strip()

        if not titulo or not enlace:
            continue

        if enlace in enlaces_encontrados:
            continue

        enlaces_encontrados.add(enlace)

        elemento = ET.SubElement(canal, "item")

        ET.SubElement(
            elemento,
            "title",
        ).text = titulo

        ET.SubElement(
            elemento,
            "link",
        ).text = enlace

        ET.SubElement(
            elemento,
            "description",
        ).text = descripcion or titulo

        ET.SubElement(
            elemento,
            "category",
        ).text = "Notas de prensa"

        identificador = ET.SubElement(
            elemento,
            "guid",
        )

        identificador.set("isPermaLink", "true")
        identificador.text = enlace

        if fecha:
            ET.SubElement(
                elemento,
                "pubDate",
            ).text = fecha

        noticias_incluidas += 1

    if noticias_incluidas == 0:
        raise RuntimeError(
            "No se pudo incluir ninguna noticia de Fluidra"
        )

    ET.indent(rss, space="  ")

    ET.ElementTree(rss).write(
        OUTPUT_FILE,
        encoding="utf-8",
        xml_declaration=True,
    )

    print(
        f"RSS creada correctamente con "
        f"{noticias_incluidas} noticias"
    )


def main():
    noticias = descargar_noticias()
    crear_rss(noticias)


if __name__ == "__main__":
    main()
