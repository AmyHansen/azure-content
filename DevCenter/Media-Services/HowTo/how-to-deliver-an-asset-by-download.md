<properties linkid="develop-media-services-how-to-guides-deliver-media-assets" urlDisplayName="Delivering Media Assets" pageTitle="How to Deliver Media Assets - Windows Azure" metaKeywords="" description="Learn about options for delivering media assets that have been uploaded to Media Services in Windows Azure. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Deliver an Asset by Download" authors=""  solutions="" writer="migree" manager="" editor=""  />





<h1>How to: Deliver an Asset by Download</h1>
This article is one in a series introducing Windows Azure Media Services programming. The previous topic was [How to: Manage Assets](http://go.microsoft.com/fwlink/?LinkID=301815&clcid=0x409).

This topic discusses options for delivering media assets uploaded to Media Services. You can deliver Media Services content in numerous application scenarios. You can download media assets, or access them by using a locator. You can send media content to another application or to another content provider. For improved performance and scalability, you can also deliver content by using a Content Delivery Network (CDN), such as the Windows Azure CDN.

This example shows how to download media assets from Media Services. The code queries the jobs associated with the Media Services account by job ID and accesses its **OutputMediaAssets** collection (which is the set of one or more output media assets that results from running a job). This  example shows how to download output media assets from a job, but you can apply the same approach to download other assets.

<pre><code> 
// Download the output asset of the specified job to a local folder.
static IAsset DownloadAssetToLocal( string jobId, string outputFolder)
{
    // This method illustrates how to download a single asset. 
    // However, you can iterate through the OutputAssets
    // collection, and download all assets if there are many. 

    // Get a reference to the job. 
    IJob job = GetJob(jobId);

    // Get a reference to the first output asset. If there were multiple 
    // output media assets you could iterate and handle each one.
    IAsset outputAsset = job.OutputMediaAssets[0];

	// Create a SAS locator to download the asset
    IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
    ILocator locator = _context.Locators.CreateSasLocator(outputAsset, accessPolicy);

    BlobTransferClient blobTransfer = new BlobTransferClient
    {
        NumberOfConcurrentTransfers = 20,
        ParallelTransferThreadCount = 20
    };

    var downloadTasks = new List&lt;Task&gt;();
    foreach (IAssetFile outputFile in outputAsset.AssetFiles)
    {
        // Use the following event handler to check download progress.
        outputFile.DownloadProgressChanged += DownloadProgress;

        string localDownloadPath = Path.Combine(outputFolder, outputFile.Name);

        Console.WriteLine("File download path:  " + localDownloadPath);

        downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));

        outputFile.DownloadProgressChanged -= DownloadProgress;
    }

    Task.WaitAll(downloadTasks.ToArray());

    return outputAsset;
}

static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
{
    Console.WriteLine(string.Format("{0} % download progress. ", e.Progress));
}
</code></pre>

For more information about delivering assets, see:
<ul>
<li><a href="http://msdn.microsoft.com/en-us/library/jj129575.aspx">Deliver Assets with the Media Services for .NET</a></li>
<li><a href="http://msdn.microsoft.com/en-us/library/jj129578.aspx">Deliver Assets with the Media Services REST API</a></li>
</ul>

<h2>Next Steps</h2>
This topic explained downloading an asset from Windows Azure Storage. For information on other ways to deliver assets go to the [How to Deliver Streaming Content](http://go.microsoft.com/fwlink/?LinkID=301942) topic.