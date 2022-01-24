using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace _3DES
{
    public partial class Form1 : Form
    {        
        private const int sizeBlock = 128; //в DES размер блока 64 бит, но поскольку в unicode символ в два раза длинее, то увеличим блок тоже в два раза
        private const int sizeChar = 16; //размер одного символа 

        private const int shiftKey = 2; //сдвиг ключа 

        private const int quantityRounds = 16; //количество раундов

        String[] blocks; //блоки в двоичном формате

        String str = ""; //введенное сообщение
        String key1 = ""; //певый ключ
        String key2 = ""; //второй ключ
        String key3 = ""; //третий ключ
        String key1Temp = ""; //для хранения первого ключа 
        String key2Temp = ""; //для хранения второго ключа
        String key3Temp = ""; //для хранения третьего ключа
        String keyTemp = "";

        public Form1()
        {
            InitializeComponent();            
        }

        /*Шифрование сообщения*/

        private void button1_Click(object sender, EventArgs e) 
        {
            String result = "";
            readText();            

            /*Шифрование первым ключом*/

            key1 = keyGen();
            result = encode(key1, str);
            key1Temp = keyTemp;

            /*Дешифрование вторым ключом*/

            key2 = keyGen();
            result = encode(key2, result);
            key2Temp = keyTemp;

            /*Шифрование третьим ключом*/

            key3 = keyGen();
            result = encode(key3, result);
            key3Temp = keyTemp;

            /*Вывод результата*/

            textBox2.Text = result;
        }

        /*Шифрование с помощью ключа исходной информации*/

        private string encode(String key, String str)
        {
            String result = "";

            cutStringBlocks(str);

            key = correctKeyWord(key, str.Length / (2 * blocks.Length));
            key = stringBinaryFormat(key);

            for (int j = 0; j < quantityRounds; j++)
            {
                for (int i = 0; i < blocks.Length; i++)
                    blocks[i] = encodeDESRound(blocks[i], key);

                key = keyNextRound(key);
            }

            key = keyPrevRound(key);
            keyTemp = stringBinaryNormalFormat(key);

            for (int i = 0; i < blocks.Length; i++)
            {
                result += blocks[i];
            }

            return stringBinaryNormalFormat(result);
        }

        /*Дешифрование при помощи ключа зашифрованного сообщения*/

        private string decode(String key, String keyTemp, String str)
        {
            String result = "";

            key = stringBinaryFormat(keyTemp);

            str = stringBinaryFormat(str);
            cutBinaryStringBlocks(str);

            for (int j = 0; j < quantityRounds; j++)
            {
                for (int i = 0; i < blocks.Length; i++)
                    blocks[i] = decodeDESRound(blocks[i], key);

                key = keyPrevRound(key);
            }

            key = keyNextRound(key);
            keyTemp = stringBinaryNormalFormat(key);

            result = "";
            for (int i = 0; i < blocks.Length; i++)
                result += blocks[i];

            return stringBinaryNormalFormat(result);
        }

        /*Дешифрование сообщения*/

        private void button2_Click(object sender, EventArgs e) 
        {
            readText();
            String result = "";

            /*Дешифрование третьим ключом*/

            result = decode(key3, key3Temp, str);
            key3Temp = keyTemp;

            /*Шифрование вторым ключом*/

            result = decode(key2, key2Temp, result);
            key2Temp = keyTemp;

            /*Дешифрование первым ключом*/

            result = decode(key1, key1Temp, result);
            key1Temp = keyTemp;

            /*Вывод результата*/

            textBox2.Text = result;   
        }

        /*Считывание сообщения из текстбокса*/

        private void readText()
        {
            str = textBox1.Text.ToString();
            if (str == "")
            {
                MessageBox.Show("Введите сообщение, которое необходимо зашифровать!");
            }
        }

        /*Генерация ключа*/

        private string keyGen()
        {
            Random random = new Random();
            return random.Next(10000, 100000).ToString();
        }

        /* Метод, доводящий строку до такого размера, чтобы она делилась на sizeBlock. 
         * Размер увеличивается с помощью добавления к исходной строке символа "#"
         */

        private string stringRightLength(string input) 
        {
            while (((input.Length * sizeChar) % sizeBlock) != 0)
                input += "#";

            return input;
        }

        /*Метод, разбивающий строку в обычном (символьном) формате на блоки*/

        private void cutStringBlocks(string input) 
        {
            blocks = new string[(input.Length * sizeChar) / sizeBlock];

            int lengthBlock = input.Length / blocks.Length;

            for (int i = 0; i < blocks.Length; i++)
            {
                blocks[i] = input.Substring(i * lengthBlock, lengthBlock);
                blocks[i] = stringBinaryFormat(blocks[i]);
            }
        }

        /*Метод, разбивающий строку в двоичном формате на блоки*/

        private void cutBinaryStringBlocks(string input) 
        {
            blocks = new string[input.Length / sizeBlock];

            int lengthBlock = input.Length / blocks.Length;

            for (int i = 0; i < blocks.Length; i++)
                blocks[i] = input.Substring(i * lengthBlock, lengthBlock);
        }

        /*Метод, переводящий строку в двоичный формат*/

        private string stringBinaryFormat(string input) 
        {
            string output = "";

            for (int i = 0; i < input.Length; i++)
            {
                string charBinary = Convert.ToString(input[i], 2);

                while (charBinary.Length < sizeChar)
                    charBinary = "0" + charBinary;

                output += charBinary;
            }

            return output;
        }

        /*Метод, доводящий длину ключа до нужной длины*/

        private string correctKeyWord(string input, int lengthKey) 
        {
            if (input.Length > lengthKey)
                input = input.Substring(0, lengthKey);
            else
                while (input.Length < lengthKey)
                    input = "0" + input;

            return input;
        }

        /*Один раунд шифрования алгоритмом DES*/

        private string encodeDESRound(string input, string key) 
        {
            string L = input.Substring(0, input.Length / 2);
            string R = input.Substring(input.Length / 2, input.Length / 2);

            return (R + XOR(L, f(R, key)));
        }

        /*Один раунд расшифровки алгоритмом DES*/

        private string decodeDESRound(string input, string key) 
        {
            string L = input.Substring(0, input.Length / 2);
            string R = input.Substring(input.Length / 2, input.Length / 2);

            return (XOR(f(L, key), R) + L);
        }

        /*XOR двух строк с двоичными данными*/

        private string XOR(string s1, string s2) 
        {
            string result = "";

            for (int i = 0; i < s1.Length; i++)
            {
                bool a = Convert.ToBoolean(Convert.ToInt32(s1[i].ToString()));
                bool b = Convert.ToBoolean(Convert.ToInt32(s2[i].ToString()));

                if (a ^ b)
                    result += "1";
                else
                    result += "0";
            }
            return result;
        }

        /*Шифрующая функция f*/

        private string f(string s1, string s2) 
        {
            return XOR(s1, s2);
        }

        /*Вычисление ключа для следующего раунда шифрования DES*/

        private string keyNextRound(string key) 
        {
            for (int i = 0; i < shiftKey; i++)
            {
                key = key[key.Length - 1] + key;
                key = key.Remove(key.Length - 1);
            }

            return key;
        }

        /*Вычисление ключа для следующего раунда расшифровки DES*/

        private string keyPrevRound(string key) 
        {
            for (int i = 0; i < shiftKey; i++)
            {
                key = key + key[0];
                key = key.Remove(0, 1);
            }

            return key;
        }

        /*Метод, переводящий строку с двоичными данными в символьный формат*/

        private string stringBinaryNormalFormat(string input) 
        {
            string output = "";

            while (input.Length > 0)
            {
                string char_binary = input.Substring(0, sizeChar);
                input = input.Remove(0, sizeChar);

                int a = 0;
                int degree = char_binary.Length - 1;

                foreach (char c in char_binary)
                    a += Convert.ToInt32(c.ToString()) * (int)Math.Pow(2, degree--);

                output += ((char)a).ToString();
            }

            return output;
        }
        
    }
}
