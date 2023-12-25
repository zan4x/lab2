using CRCcs;
using System;
using ConsoleTables;
using System.Linq;

namespace crcProgram
{
    class MainProgram
    {
        // Главная функция программы
        static unsafe void Main(string[] args)
        {
            // Инициализация переменных
            int bitCount = 32;
            uint newCrc32;
            uint newCrc16;

            // Создание и инициализация массивов счетчиков
            int[] countCrc16 = Enumerable.Repeat(0, bitCount).ToArray();
            int[] countCrc32 = Enumerable.Repeat(0, bitCount).ToArray();
            int[] countXor = Enumerable.Repeat(0, bitCount).ToArray();
            int[] totalCount = Enumerable.Repeat(0, bitCount).ToArray();

            int index;
            int dataSize = 128;

            byte[] data = new byte[dataSize];

            Console.WriteLine("Initializing data");
            // Заполнение массива данными и вывод на консоль
            for (index = 0; index < dataSize; index++)
            {
                data[index] = (byte)index;
                Console.Write($"{data[index]:X02} ");
                if ((index + 1) % 32 == 0)
                    Console.WriteLine();
            }
            Console.WriteLine();

            fixed (byte* ptr8 = data)
            {
                long timeStart, timeEnd;
                uint* ptr32 = (uint*)ptr8;
                uint initialVal = ptr32[0];

                // Вычисление CRC и XOR для данных
                uint crc32 = CRC.crc32b(data);
                uint crc16 = CRC.Crc16Ccitt(data);
                int xorVal = data.Aggregate(0, (current, value) => current ^ value);
                timeStart = Environment.TickCount;
                for (uint x = 1; x <= 0xFFFFFF; x++)
                {
                    // Подсчет количества единиц в числе
                    uint oneCount = CountOnes(x);
                    ptr32[0] = initialVal ^ (uint)x;

                    // Вычисление новых значений CRC и XOR
                    newCrc32 = CRC.crc32b(data);
                    newCrc16 = CRC.Crc16Ccitt(data);

                    int newXorVal = data.Aggregate(0, (current, value) => current ^ value);

                    // Обновление счетчиков
                    if (newXorVal == xorVal)
                        countXor[oneCount]++;
                    if (newCrc16 == crc16)
                        countCrc16[oneCount]++;
                    if (newCrc32 == crc32)
                        countCrc32[oneCount]++;
                    totalCount[oneCount]++;

                    // Вывод прогресса выполнения
                    if (x % 1000 == 0)
                    {
                        timeEnd = Environment.TickCount;
                        if (timeEnd - timeStart > 1000)
                        {
                            Console.Write($"\r процент подсчёта  {(x * 100.0) / 0xFFFFFF:0.#} ");
                            timeStart = timeEnd;
                        }
                    }
                }

                // Вывод результатов в таблицу
                Console.WriteLine("\n{0,5} {1,5} {2,5} {3,5} {4,5}", "№бита ", "N", "xor", "crc16", "crc32");
                for (uint i = 1; i < bitCount; i++)
                {
                    Console.WriteLine("{0,5} {1,5} {2,5} {3,5} {4,5}", i, totalCount[i], countXor[i], countCrc16[i], countCrc32[i]);
                }
            }
        }

        // Функция для подсчета количества единиц в двоичном представлении числа
        private static uint CountOnes(uint number)
        {
            uint oneCount = 0;

            while (number != 0)
            {
                oneCount += number & 1;
                number >>= 1;
            }

            return oneCount;
        }
    }
}
