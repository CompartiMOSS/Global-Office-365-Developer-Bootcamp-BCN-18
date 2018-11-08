## Segunda prueba, subir un documento a OneDrive, convertirlo a PDF y enviarlo por mail

1. En la clase **GraphSdkHelper** añade los siguientes métodos

```csharp
public static async Task SendEmail(GraphServiceClient graphClient, IHostingEnvironment hostingEnvironment, string recipients, HttpContext httpContext)
        {
            if (recipients == null) return;

            var attachments = new MessageAttachmentsCollectionPage();

            try
            {
                // Load user's profile picture.
                var pictureStream = await GetMyPictureStream(graphClient, httpContext);

                if (pictureStream != null)
                {

                    // Copy stream to MemoryStream object so that it can be converted to byte array.
                    var pictureMemoryStream = new MemoryStream();
                    await pictureStream.CopyToAsync(pictureMemoryStream);

                    // Convert stream to byte array and add as attachment.
                    attachments.Add(new FileAttachment
                    {
                        ODataType = "#microsoft.graph.fileAttachment",
                        ContentBytes = pictureMemoryStream.ToArray(),
                        ContentType = "image/png",
                        Name = "me.png"
                    });
                }

                var fileName = @"Bootcamp2018/temp.docx";
                List<Option> options = new List<Option>();
                options.Add(new QueryOption("format", "pdf"));

                var pdfFile = await graphClient.Me
                    .Drive
                    .Root
                    .ItemWithPath(fileName)
                    .Content
                    .Request(options)
                    .GetAsync();

                var pdfMemoryStream = new MemoryStream();
                await pdfFile.CopyToAsync(pdfMemoryStream);

                attachments.Add(new FileAttachment
                {
                    ODataType = "#microsoft.graph.fileAttachment",
                    ContentBytes = pdfMemoryStream.ToArray(),
                    ContentType= "application/pdf",
                    Name = "doc.pdf"
                });
            }
            catch (Exception e)
            {
                switch (e.Message)
                {
                    case "ResourceNotFound":
                        break;
                    default:
                        throw;
                }
            }

            // Prepare the recipient list.
            var splitRecipientsString = recipients.Split(new[] { ";" }, StringSplitOptions.RemoveEmptyEntries);
            var recipientList = splitRecipientsString.Select(recipient => new Recipient
            {
                EmailAddress = new EmailAddress
                {
                    Address = recipient.Trim()
                }
            }).ToList();

            // Build the email message.
            var email = new Message
            {
                Body = new ItemBody
                {
                    //Content = System.IO.File.ReadAllText(hostingEnvironment.WebRootPath + "/email_template.html"),
                    //ContentType = BodyType.Html,
                },
                Subject = "Sent from the Microsoft Graph Connect sample",
                ToRecipients = recipientList,
                Attachments = attachments
            };

            await graphClient.Me.SendMail(email, true).Request().PostAsync();
        }

        public static async Task OneDriveUpload(GraphServiceClient graphClient, Stream stream, HttpContext httpContext)
        {
            var fileName = @"Bootcamp2018/temp.docx";

            await graphClient.Me
                .Drive
                .Root
                .ItemWithPath(fileName)
                .Content
                .Request()
                .PutAsync<DriveItem>(stream);
        }
```

2. Dentro del controlador **Home** añade el siguiente método

```csharp
[HttpPost("UploadFiles")]
        public async Task<IActionResult> Post(IFormFile file)
        {
            if (User.Identity.IsAuthenticated)
            {

                // Get user's id for token cache.
                var identifier = User.FindFirst(Startup.ObjectIdentifierType)?.Value;
                

                long size = file.Length;

                // full path to file in temp location
                var filePath = Path.GetTempFileName();


                if (size > 0)
                {
                    using (var stream = new FileStream(filePath, FileMode.Create))
                    {
                        await file.CopyToAsync(stream);
                    }

                    using (FileStream fileStream = new FileStream(filePath, FileMode.Open))
                    {
                        // Initialize the GraphServiceClient.
                        var graphClient = _graphSdkHelper.GetAuthenticatedClient(identifier);

                        await GraphService.OneDriveUpload(graphClient, fileStream, HttpContext);

                        var email = User.FindFirst("preferred_username")?.Value;
                        await GraphService.SendEmail(graphClient, _env, email, HttpContext);
                    }

                }

                // process uploaded files
                // Don't rely on or trust the FileName property without validation.

                return View();
            }
            else
            {
                return Redirect("/");
            }
        }
```

3. En la vista Index.cshtml añade el siguiente código al final del documento, dentro de ```@if csharp(User.Identity.IsAuthenticated)```

```html
    <br />
    <form method="post" enctype="multipart/form-data" asp-controller="UploadFiles" asp-action="Index">
        <div class="form-group">
            <div class="col-md-10">
                <p>Sube un documento de word:</p>
                <input type="file" name="file" />
            </div>
        </div>
        <div class="form-group">
            <div class="col-md-10">
                <input type="submit" value="Upload" />
            </div>
        </div>
    </form>
```
   