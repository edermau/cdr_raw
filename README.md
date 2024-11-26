Estrutura de um Registro CDR
Cada registro CDR possui os seguintes campos e tamanhos:

Campo	Tamanho (bytes)	Descrição
MSISDN	11	Número de telefone do assinante (ASCII).
IMSI	15	Identificação única do assinante móvel.
IMEI	15	Identificação única do dispositivo móvel.
Start Time	4	Horário de início da chamada (UNIX timestamp).
End Time	4	Horário de término da chamada (UNIX timestamp).
Call Duration	4	Duração da chamada, em segundos.
Call Type	1	Tipo de evento (1=VOICE, 2=SMS, 3=DATA).
Data Volume	4	Volume de dados usados, em bytes.
Cada registro tem um total de 59 bytes.

Analisando o Arquivo com Hexdump
1. Visualizar o Arquivo
Use o comando hexdump para inspecionar o arquivo binário:

bash
Copiar código
hexdump -C -n 59 cdr_amostras_raw.bin
Saída Exemplo
bash
Copiar código
00000000  31 31 39 38 37 36 35 34  33 32 31 33 31 30 32 36  |1198765432131026|
00000010  30 30 30 30 30 30 30 30  33 35 36 39 33 38 30 33  |0000000035693803|
00000020  35 36 34 33 38 30 39 5f  8a 93 00 5f 8a 94 00 00  |5643809_..._....|
00000030  0e 10 01 00 00 04 00                              |.......          |
2. Identificando os Campos
Com base no layout do registro:

Campo	Bytes	Valor Exemplo
MSISDN	0-10	31 31 39 38 37 36 35 34 33 32 31 → 11987654321
IMSI	11-25	33 31 30 32 36 30 30 30 30 30 30 30 30 30 → 310260000000000
IMEI	26-40	33 35 36 39 33 38 30 33 35 36 34 33 38 30 39 → 356938035643809
Start Time	41-44	5f 8a 93 00 → 1609459200 (01/01/2021 00:00:00 UTC)
End Time	45-48	5f 8a 94 00 → 1609459260 (01/01/2021 00:01:00 UTC)
Call Duration	49-52	00 0e 10 → 3600 segundos
Call Type	53	01 → VOICE
Data Volume	54-58	00 04 00 → 1024 bytes (1 KB)
Extraindo Campos Específicos
Use o comando hexdump para extrair campos específicos:

MSISDN (11 bytes):

bash
Copiar código
hexdump -s 0 -n 11 -e '11/1 "%_p\n"' cdr_amostras_raw.bin
IMSI (15 bytes):

bash
Copiar código
hexdump -s 11 -n 15 -e '15/1 "%_p\n"' cdr_amostras_raw.bin
Start Time (4 bytes):

bash
Copiar código
hexdump -s 41 -n 4 -e '"%d\n"' cdr_amostras_raw.bin
Data Volume (4 bytes):

bash
Copiar código
hexdump -s 54 -n 4 -e '"%d\n"' cdr_amostras_raw.bin
Automação com Script Python
Use o script abaixo para decodificar os registros completos de forma automatizada:

python
Copiar código
import struct

# Tamanho do registro e formato
RECORD_SIZE = 59
FORMAT = '>11s15s15sIIIIB'

def decode_cdr(file_path):
    with open(file_path, "rb") as file:
        print(f"{'MSISDN':<15} {'Start Time':<15} {'End Time':<15} {'Call Type':<10}")
        print("=" * 60)
        while chunk := file.read(RECORD_SIZE):
            msisdn, imsi, imei, start_time, end_time, call_duration, call_type = struct.unpack(FORMAT, chunk)
            print(f"{msisdn.decode().strip():<15} {start_time:<15} {end_time:<15} {call_type:<10}")

decode_cdr("cdr_amostras_raw.bin")
Saída Esperada
bash
Copiar código
MSISDN         Start Time      End Time        Call Type
============================================================
11987654321    1609459200      1609459260      1
Referências
Hexdump Documentation
Formato de Timestamps Unix
