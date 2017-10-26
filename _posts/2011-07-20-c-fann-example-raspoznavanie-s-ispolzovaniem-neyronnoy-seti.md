---
layout: post
title: C# FANN example &#8212; распознавание с использованием нейронной сети

tags: [ai, fann, neural, neuralnet]
---

http://leenissen.dk/fann/wp/ - сайт библиотеки FANN

http://code.google.com/p/fanndotnetwrapper/ - обертка под .NET

http://leenissen.dk/fann/fann.html - php

http://code.google.com/p/fanntool/ - тул по работе с библиотекой FANN - самое классное в нем - что он может подобрать оптимальные параметры нейронной сети.

В примере стандартная задача по операции xor, и еще одна по определению прямоугольника - на вход подается сто бит (грубо говоря рисунок - 10х10 пикселей) - сеть пытается определить нарисован ли на ней прямоугольник.

**Пример:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using FANN.Net;
using System.IO;

namespace fann
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("XOR");
            NeuralNet xornet = xor.get_xor_net();
            xor.test(xornet, 0, 0);
            xor.test(xornet, 0, 1);
            xor.test(xornet, 1, 0);
            xor.test(xornet, 1, 1);

            Console.WriteLine("\nBOX");

            NeuralNet boxnet = box.get_box_net();
            box.test(boxnet, box.randombox());
            box.test(boxnet, box.randombox());
            box.test(boxnet, box.randombox_with_some_elements());
            box.test(boxnet, box.randombox_without_some_elements());
            box.test(boxnet, box.rand());

            Console.ReadKey();
        }
    }

    class xor
    {
        public static NeuralNet get_xor_net()
        {
            NeuralNet net = new NeuralNet();

            if (!File.Exists("xor.net"))
            {
                if (!File.Exists("xor.data"))
                {
                    create_training_file();
                    Console.WriteLine("xor.data generated");
                }

                List<uint> layers = new List<uint>();
                layers.Add(2); // inputs
                layers.Add(3); // hidden
                layers.Add(1); // output

                net.CreateStandardArray(layers.ToArray());

                net.SetLearningRate((float)0.7);

                net.SetActivationSteepnessHidden(1.0);
                net.SetActivationSteepnessOutput(1.0);

                net.SetActivationFunctionHidden(ActivationFunction.SigmoidSymmetric);
                net.SetActivationFunctionOutput(ActivationFunction.SigmoidSymmetric);

                net.SetTrainStopFunction(StopFunction.Bit);
                net.SetBitFailLimit(0.01f);
                // Set additional properties such as the training algorithm
                //net.SetTrainingAlgorithm(FANN::TRAIN_QuickProp);

                TrainingData data = new TrainingData();
                data.ReadTrainFromFile("xor.data");

                net.InitWeights(data);

                net.TrainOnData(data,
                    1000, // max iterations
                    10, // iterations between report
                    0 //desired error
                    );

                net.Save("xor.net");
            }

            net.CreateFromFile("xor.net");

            return net;
        }

        public static void create_training_file()
        {
            StringBuilder sb = new StringBuilder();
            sb.AppendLine("4 2 1");
            sb.AppendLine("0 0");
            sb.AppendLine("1");
            sb.AppendLine("0 1");
            sb.AppendLine("0");
            sb.AppendLine("1 0");
            sb.AppendLine("0");
            sb.AppendLine("1 1");
            sb.AppendLine("0");
            File.WriteAllText("xor.data", sb.ToString());
        }

        public static void test(NeuralNet net, double a, double b)
        {
            double r = net.Run(new double[] { a, b })[0];
            r = Math.Round(r, 1);
            Console.WriteLine(string.Format("{0} xor {1} = {2}", a, b, r));
        }
    }

    class box
    {
        public const int WIDTH = 10;

        public static NeuralNet get_box_net()
        {
            NeuralNet net = new NeuralNet();

            if (!File.Exists("box.net"))
            {
                if (!File.Exists("box.data"))
                {
                    box.create_training_file();
                    Console.WriteLine("box.data generated");
                }

                List<uint> layers = new List<uint>();
                layers.Add(100); // inputs
                layers.Add(300); // hidden
                layers.Add(1); // output

                net.CreateStandardArray(layers.ToArray());

                net.SetLearningRate((float)0.7);

                net.SetActivationSteepnessHidden(1.0);
                net.SetActivationSteepnessOutput(1.0);

                net.SetActivationFunctionHidden(ActivationFunction.SigmoidSymmetric);
                net.SetActivationFunctionOutput(ActivationFunction.SigmoidSymmetric);

                net.SetTrainStopFunction(StopFunction.Bit);
                net.SetBitFailLimit(0.01f);
                // Set additional properties such as the training algorithm
                //net.SetTrainingAlgorithm(FANN::TRAIN_QuickProp);

                TrainingData data = new TrainingData();
                data.ReadTrainFromFile("box.data");

                net.InitWeights(data);

                net.TrainOnData(data,
                    1000, // max iterations
                    10, // iterations between report
                    0 //desired error
                    );

                net.Save("box.net");
            }

            net.CreateFromFile("box.net");

            return net;
        }

        public static void create_training_file()
        {
            File.WriteAllText("box.data", "5000 100 1" + Environment.NewLine);

            for (int i = 0; i < 1000; i++)
            {
                File.AppendAllText("box.data", box.tostring(box.randombox(), true));
                File.AppendAllText("box.data", box.tostring(box.randombox_without_some_elements(), false));
                File.AppendAllText("box.data", box.tostring(box.randombox_with_some_elements(), false));
                File.AppendAllText("box.data", box.tostring(box.rand(), false));
                File.AppendAllText("box.data", box.tostring(box.randombox(), true));
            }
        }

        public static int[,] rand()
        {
            int[,] input = get_empty();

            Random rnd = new Random();

            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    input[x, y] = (rnd.Next(0, 1000) > 500) ? 1 : 0;
                }
            }

            return input;
        }

        public static int[,] randombox_with_some_elements()
        {
            int[,] input = randombox();

            Random rnd = new Random();

            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    if (rnd.Next(0, 1000) > 500)
                    {
                        input[x, y] = 1;
                    }
                }
            }

            return input;
        }

        public static int[,] randombox_without_some_elements()
        {
            int[,] input = randombox();

            Random rnd = new Random();

            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    if (input[x, y] == 1 && rnd.Next(0, 1000) > 500)
                    {
                        input[x, y] = 0;
                    }
                }
            }

            return input;
        }

        public static int[,] randombox()
        {
            int[,] input = get_empty();

            Random rnd = new Random();

            int x1 = rnd.Next(0, WIDTH - 3);
            int y1 = rnd.Next(0, WIDTH - 3);

            int x2 = rnd.Next(x1 + 2, WIDTH - 1);
            int y2 = rnd.Next(y1 + 2, WIDTH - 1);

            for (int x = x1; x <= x2; x++)
            {
                input[x, y1] = 1;
                input[x, y2] = 1;
            }

            for (int y = y1; y <= y2; y++)
            {
                input[x1, y] = 1;
                input[x2, y] = 1;
            }

            return input;
        }

        public static void print(int[,] input)
        {
            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    Console.Write(input[x, y]);
                    Console.Write(" ");
                }
                Console.WriteLine();
            }
        }

        public static string tostring(int[,] input, bool isbox)
        {
            List<string> items = new List<string>();
            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    items.Add(input[x, y].ToString());
                }
            }

            string b = isbox ? "1" : "0";

            return string.Join(" ", items.ToArray()) + Environment.NewLine + b + Environment.NewLine;
        }

        public static int[,] get_empty()
        {
            int[,] input = new int[WIDTH, WIDTH];

            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    input[x, y] = 0;
                }
            }
            return input;
        }

        public static void test(NeuralNet net, int[,] input)
        {
            List<double> items = new List<double>();

            for (int x = 0; x < WIDTH; x++)
            {
                for (int y = 0; y < WIDTH; y++)
                {
                    items.Add((double)input[x, y]);
                }
            }

            double r = net.Run(items.ToArray())[0];
            r = Math.Round(r, 1);

            print(input);
            Console.WriteLine((r == 1) ? "IS box" : "NOT box");
        }
    }

    class Example
    {
        #region example from sources
        // Test function that demonstrates usage of the fann C++ wrapper
        private static void xor_test()
        {
            System.Console.WriteLine("XOR test started.");

            const float LearningRate = 0.7f;
            const uint numInput = 2;
            const uint numHidden = 3;
            const uint numOutput = 1;
            const float desired_error = 0;
            const uint max_iterations = 1000;
            const uint iterations_between_reports = 10;

            System.Console.WriteLine("Creating network.");

            NeuralNet net = new NeuralNet();

            List<uint> layers = new List<uint>();
            layers.Add(numInput);
            layers.Add(numHidden);
            layers.Add(numOutput);

            net.CreateStandardArray(layers.ToArray());

            net.SetLearningRate(LearningRate);

            net.SetActivationSteepnessHidden(1.0);
            net.SetActivationSteepnessOutput(1.0);

            net.SetActivationFunctionHidden(ActivationFunction.SigmoidSymmetric);
            net.SetActivationFunctionOutput(ActivationFunction.SigmoidSymmetric);

            net.SetTrainStopFunction(StopFunction.Bit);
            net.SetBitFailLimit(0.01f);
            // Set additional properties such as the training algorithm
            //net.SetTrainingAlgorithm(FANN::TRAIN_QuickProp);

            // Output network type and parameters
            System.Console.WriteLine("Network Type                         :  ");
            switch (net.GetNetworkType())
            {
                case NetworkType.Layer:
                    System.Console.WriteLine("LAYER");
                    break;
                case NetworkType.ShortCut:
                    System.Console.WriteLine("SHORTCUT");
                    break;
                default:
                    System.Console.WriteLine("UNKNOWN");
                    break;
            }
            net.PrintParameters();

            System.Console.WriteLine("Training network.");

            TrainingData data = new TrainingData();

            if (data.ReadTrainFromFile("xor.data"))
            {
                // Initialize and train the network with the data
                net.InitWeights(data);

                System.Console.WriteLine("Max Epochs " + max_iterations + ". "
                    + "Desired Error: " + desired_error);
                /*
                net.Callback += (nn, train, max_epochs, epochs_between_reports, de, epochs)
                    => {
                        System.Console.WriteLine("Epochs     " + epochs + ". " + "Current Error: " + nn.GetMSE() + "\n");
                        return 0;
                    };
                */
                net.TrainOnData(data, max_iterations,
                    iterations_between_reports, desired_error);

                System.Console.WriteLine("Testing network.");

                for (uint i = 0; i < data.TrainingDataLength; ++i)
                {
                    // Run the network on the test data
                    double calcOut = net.Run(data.Input[i])[0];

                    System.Console.WriteLine("XOR test (" + data.Input[i][0] + ", "
                            + data.Input[i][1] + ") -> " + calcOut
                            + ", should be " + data.Output[i][0] + ", "
                            + "difference = "
                            + Math.Abs(calcOut - data.Output[i][0]));
                }

                System.Console.WriteLine("Saving network.");

                // Save the network in floating point and fixed point
                net.Save("xor_float.net");
                uint decimal_point = (uint)net.SaveToFixed("xor_fixed.net");
                data.SaveTrainToFixed("xor_fixed.data", decimal_point);

                System.Console.WriteLine("XOR test completed.");

            }
        }
        #endregion
    }

}
```

**На выходе дает такое:**

```
XOR
0 xor 0 = 1
0 xor 1 = 0
1 xor 0 = 0
1 xor 1 = 0

BOX
0 0 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 0 0
1 0 0 0 0 0 0 1 0 0
1 0 0 0 0 0 0 1 0 0
1 1 1 1 1 1 1 1 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
IS box
0 0 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 0 0
1 0 0 0 0 0 0 1 0 0
1 0 0 0 0 0 0 1 0 0
1 1 1 1 1 1 1 1 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
IS box
0 0 0 1 0 0 1 1 0 0
1 1 1 1 1 1 1 1 1 0
1 0 0 0 1 0 1 1 0 0
1 1 1 0 1 1 0 1 1 0
1 1 1 1 1 1 1 1 1 1
1 0 0 1 1 1 1 1 0 0
1 1 1 1 0 1 0 1 1 0
1 1 1 1 0 0 1 0 1 1
1 0 1 1 1 1 1 1 1 1
1 0 1 1 0 1 1 1 1 1
NOT box
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 1 0 0 0
0 0 0 0 1 0 0 0 0 0
0 0 0 0 1 0 0 0 0 0
0 0 0 0 0 1 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
NOT box
1 1 0 0 1 0 1 1 0 1
0 1 1 1 1 1 0 0 0 0
1 0 1 0 0 0 1 1 1 0
1 0 0 1 0 0 0 1 1 0
0 1 1 1 1 0 0 1 0 1
1 0 1 1 1 0 1 0 0 0
1 0 0 0 0 0 0 0 0 1
0 0 1 1 0 1 1 0 0 0
0 1 0 1 1 0 0 0 1 1
0 0 1 0 0 0 0 1 0 1
NOT box
```

**Пример на php из исходников pecl модуля** (сохраняю на память):

```php
<?php

/* demo.php
    * $Id: demo.php,v 1.3 2004/01/19 22:33:51 tadpole9 Exp $
    *
    * This file should help explain the FANN API from a PHP perspective. It
    * is basically a PHP port of the simple_test/simple_train examples from
    * the FANN distribution. The original C versions can be found in
    * $FANN_DIR/examples
    *
    * There are a few functions that aren't demonstrated in this file, here
    * are their prototypes:
    *
    * void fann_randomize_weights(resource ann [, double min [, double max]])
    * void fann_set_learning_rate(resource ann, float learning_rate)
    * void fann_set_activation_function_hidden(resource ann, int activation_function)
    * void fann_set_activation_function_output(resource ann, int activation_function)
    * void fann_set_activation_hidden_steepness(resource ann, int activation_function)
    * void fann_set_activation_output_steepness(resource ann, int activation_function)
    * double fann_get_MSE(resource ann)
    * double fann_get_learning_rate(resource ann)
    * long fann_get_num_input(resource ann)
    * long fann_get_num_output(resource ann)
    * long fann_get_activation_function_hidden(resource ann)
    * long fann_get_activation_function_output(resource ann)
    * double fann_get_activation_hidden_steepness(resource ann)
    * double fann_get_activation_output_steepness(resource ann)
    * long fann_get_total_neurons(resource ann)
    * long fann_get_total_connections(resource ann)
    *
    * If you have any questions or comments, please e-mail Evan Nemerson
    * <evan@coeus-group.com>
    */

/* If you don't want to compile FANN into PHP... */
if ( !extension_loaded('fann') ) {
    if ( !dl('fann.so') ) {
    exit("You must install the FANN extension. You can get it from http://fann.sf.net/\n");
    }
}

/* Create an artificial neural network */
$ann = fann_create(
            array(2, 4, 1), /* layers. in this case, three layers- two input neurons, 4 neurons in a hidden layer, and one output neuron */
            1.0, /* learning rate */
            0.7 /* connection rate */
            );

/* To load from a file, you can use. If your version of PHP includes the streams API (4.3.0+ ?),
    * this can be anything accessible through streams (http, ftp, https, etc) */
// $ann = fann_create("http://example.com/xor_float.net");

/* Train the network using the same data as is in the xor.data file */
fann_train($ann,
        array(
            array(
                array(0,0), /* Input(s) */
                array(0) /* Output(s) */
                ),
            array(
                array(0,1), /* Input(s) */
                array(1) /* Output(s) */
                ),
            array(
                array(1,0), /* Input(s) */
                array(1) /* Output(s) */
                ),
            array(array(1,1), /* Input(s) */
                array(0) /* Output(s) */
                )
            ),
        100000, /* Maximum number of epochs */
        0.00001, /* Desired error. */
        1000 /* Number of epochs between reports */
        );

/* To achieve the same effect as the above with the data stored in an external file... Also works
    * with the streams API, when available. */
// fann_train($ann, '/home/tadpole/local/src/fann/examples/xor.data', 100000, 0.00001, 1000);

print_r(fann_run($ann, array(0,0))); // Should be ~ 0
print_r(fann_run($ann, array(0,1))); // Should be ~ 1
print_r(fann_run($ann, array(1,0))); // Should be ~ 1
print_r(fann_run($ann, array(1,1))); // Should be ~ 0

/* This function is pretty simple. It will use the streams API if available. */
fann_save($ann, 'xor_float.net');

?>
```

Наметки по установке fann на linux'е:

```sh
sudo pear install http://pecl.php.net/get/fann-0.1.1.tgz
sudo pecl install fann-devel
pecl install fann-0.1.1
```

без этого не конфигурится

```sh
sudo apt-get install libfann-dev
sudo apt-get install libfann1-dev
libfann1-dev libfann1 libfann2

wget http://pecl.php.net/get/fann-0.1.1.tgz
tar zxf fann-0.1.1.tgz
cd fann-0.1.1
sudo phpize
sudo ./configure
sudo make
sudo make install
```

после make install скажет:

```
Installing shared extensions:     /usr/lib/php5/20090626+lfs/
```

```sh
sudo touch /etc/php5/apache2/conf.d/fann.ini
fann_extension=/usr/lib/php5/20090626+lfs/fann.so
```

пока ничего не получилось... (
