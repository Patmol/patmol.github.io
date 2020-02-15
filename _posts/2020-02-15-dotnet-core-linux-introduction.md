---
title: ".NET Core - Linux Introduction"
date: 2020-02-15 14:00:00 +0800
categories: [.NET Core]
tags: [programming, tutorial, .netcore, linux]
---

When I met people outside the Microsoft ecosystem, I always have the feeling that, for them, Microsoft is still the same as in 2000. They have a not so good operating system and their tools are only available on Windows.

I've got to say to them; things have changed a lot! Since Satya Nadella is CEO of Microsoft (and a little before), Microsoft has embraced every platform. You can now run Linux on your Windows or develop .NET Application on Linux and running them on Linux, macOS, and, of course, Windows.

It's this last point that I want to write about today. In this post, we are building a small application that detects and blurs the faces of one or more pictures. If you are following the night classes at the UNamur, it can be useful :-)

## Installing .NET Core
The first step is to install .NET Core on your operating system. In this case, I'm installing it on a Debian 10, but the process may change depending, so if you are not using Debian, you can find the procedure for your OS in the [Microsoft documentation](https://docs.microsoft.com/en-us/dotnet/core/install/sdk?pivots=os-linux).

Those steps only need to be performed once.

### Register Microsoft key and feed
To install .NET Core, you, foremost, need to :
* Register the Microsoft key
* Register the product repository
* Install required dependencies

To perform those actions, run the following command in the terminal:
```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
sudo mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
wget -q https://packages.microsoft.com/config/debian/10/prod.list
sudo mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
sudo chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
sudo chown root:root /etc/apt/sources.list.d/microsoft-prod.list
```

### Install the .NET Core SDK
Once it's done, we are installing the .NET Code SDK by running the following commands:
```bash
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```

## Creation of the application
As I said, we are building a small application to blur the face on one or more images. The first step is to create a new .NET Core console project with those commands:
```bash
mkdir faceblur
cd faceblur
dotnet new console
```

Once it's done, you can try to run the program by running the command:
```bash
dotnet run
```

And an `Hello World!` message should be displayed. If the message is displayed, the application is running fine, and we can work on our application.

### Our code editor

The best code editor to use when you are working with a .NET Core project is _Visual Studio Code_. It's a free and open-source editor from Microsoft (when I've written that they are not the same, I wasn't kidding).

You can download it [here](https://code.visualstudio.com).

On my side, I'm using Vim. Not because I don't like VS Code, it's quite the opposite, but because the Debian I'm working on is installed without GUI.

### The existing code

You can open the Program.cs file with your editor, and you've got the most basic C# program.

```csharp
using System;

namespace faceblur
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World");
        }
    }
}
```

### Some preparations - The packages to play with our images

To blur our images, we need an external package that we can install with this command:
```bash
dotnet add package ImageProcessorCore --version 1.0.0-alpha1095
```

### More preparations - How to detect faces in an image

To detect the faces on an image, we are using the Cognitive Service _Face_ from Microsoft, but you can check for an alternative if you prefer. I know that Google is also providing the same kind of service.

To use the Microsoft service, you need an Azure account, and you've got 20 calls by minutes and 30,000 free transactions per month to the service, which is more than enough for our test. If you want to use it, you can find more information [here](https://azure.microsoft.com/en-us/services/cognitive-services/).

Once you've created the service, you've got a _key_ and an _endpoint_ hat you need to use in the application. And before writing code, we add the NuGet package for the Cognitive Service through this command

```bash
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.5.0-preview.1
```

### Finally, we are writing some code!

Let's start by writing code authenticate our client to the Cognitive Service.

```csharp
using System;
using Microsoft.Azure.CognitiveServices.Vision.Face;

namespace faceblur
{
    class Program
    {
        private static string SUBSCRIPTION_KEY = Environment.GetEnvironmentVariable("FACE_SUBSCRIPTION_KEY");
        private static string ENDPOINT = Environment.GetEnvironmentVariable("FACE_ENDPOINT");

        static void Main(string[] args)
        {
            IFaceClient client = Program.Authenticate(Program.ENDPOINT, Program.SUBSCRIPTION_KEY);
        }

        private static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }
    }
}
```

In our case, we are using environment variables to store our key and ou endpoint. If you want, you can directly write that information in the code, but it's not an excellent way to work, especially if you want to share your code.

You can have more information on how to use environment variables on the [Microsoft documentation](https://docs.microsoft.com/en-gb/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Clinux#configure-an-environment-variable-for-authentication).

Once it's done, we can write the code to detect and return the position of the faces on a list of images.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;
using ImageProcessorCore;
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;

namespace faceblur
{
    class Program
    {
        private static string SUBSCRIPTION_KEY = Environment.GetEnvironmentVariable("FACE_SUBSCRIPTION_KEY");
        private static string ENDPOINT = Environment.GetEnvironmentVariable("FACE_ENDPOINT");

        static async Task Main(string[] args)
        {
            var client = Program.Authenticate(Program.ENDPOINT, Program.SUBSCRIPTION_KEY);

            Console.WriteLine("Detecting faces ...");
            var facesPosition = await Program.DetectFaceExtract(client, args, RecognitionModel.Recognition02);
        }

        private static IFaceClient Authenticate(string endpoint, string key)
        {
            return new FaceClient(new ApiKeyServiceClientCredentials(key)) { Endpoint = endpoint };
        }

        private static async Task<IList<(string, IList<Rectangle>)>> DetectFaceExtract(IFaceClient client, string[] paths, string recongnitionModel)
        {                        
            var result = new List<(string, IList<Rectangle>)>();

            foreach (var path in paths)
            {
                IList<DetectedFace> detectedFaces;
                var facesToBlur = new List<Rectangle>();

                using (var stream = new FileStream(path, FileMode.Open, FileAccess.Read))
                {
                    detectedFaces = await client.Face.DetectWithStreamAsync(stream);

                    Console.WriteLine($"{detectedFaces.Count} faces detected in the image {path}");

                    foreach (var detectedFace in detectedFaces)
                    {
                        facesToBlur.Add(new Rectangle(
                            detectedFace.FaceRectangle.Left,
                            detectedFace.FaceRectangle.Top,
                            detectedFace.FaceRectangle.Width,
                            detectedFace.FaceRectangle.Height));
                    }
                }

                result.Add((path, facesToBlur));
            }

            return result;
        }
    }
}
```

Now that we have the position, we just need to blur this area. To perform this action, we are using the ImageProcessorCore library and add this method to our code.

```csharp
private static void BlurImage(IList<(string, IList<Rectangle>)> imageToBlurs)
{
    foreach (var imageToBlur in imageToBlurs)
    {
        Console.WriteLine($"Bluring image {imageToBlur.Item1} ...");

        var path = Path.GetDirectoryName(imageToBlur.Item1);
        var fileNameWithoutExtension = Path.GetFileNameWithoutExtension(imageToBlur.Item1);
        var extension = Path.GetExtension(imageToBlur.Item1);
        var blurFile = $"{path}/{fileNameWithoutExtension}-blur{extension}";

        if (File.Exists(blurFile))
        {
            File.Delete(blurFile);
        }

        using (var stream = File.OpenRead(imageToBlur.Item1))
        using (var output = File.OpenWrite(blurFile))
        {
            var image = new Image<Color, uint>(stream);
                
            foreach (var face in imageToBlur.Item2)
            {
                image = image.BoxBlur(20, face);
            }

            image.SaveAsJpeg(output);
        }

        Console.WriteLine("Image blured");
    }
}
```

And this method calls in our Main method.

```csharp
static async Task Main(string[] args)
{
    var client = Program.Authenticate(Program.ENDPOINT, Program.SUBSCRIPTION_KEY);

    Console.WriteLine("Detecting faces ...");
    var facesPosition = await Program.DetectFaceExtract(client, args, RecognitionModel.Recognition02);

    Console.WriteLine("Bluring faces ...");
    Program.BlurImage(facesPosition);
}
```

### Run our application

Now that we have developed our application, we can build it and then test it.

To build the application, run the command `dotnet build` and you should have this result.

```bash
Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:03.65
```

If the build has succeeded, you can run the application with the command.

```bash
dotnet run -- path/to/your/images
```

Like thiss
```bash
dotnet run -- ./images/image-1.png
dotnet run -- ./images/image-1.png ./images/image-2.jpg
```

And here you've got your images blurred. You can find them in the same folder.

If you want to try the code directly, this one is available on [GitHub](https://github.com/Patmol/faceblur)