# TheReadingClub-Public
I built this but it's private and for-profit. Check out the website (https://www.thereadingclubapp.com/) and download the iOS beta (https://testflight.apple.com/join/OGNZivve).

- Developed learn-to-read app (https://www.thereadingclubapp.com/) that is highly interactive and leverages 
data science tools to (1) engage kids and their parents or teachers in reading together, and (2) enable parents 
to monitor and evaluate their children’s performance with custom reports.
- Programmed back-end API (Node.js) (e.g., CRUD endpoints, data aggregation, unit tests), and front-end (React) 
(e.g., interactive data visualizations).
- Built mobile app available via Apple’s Test Flight (https://testflight.apple.com/join/OGNZivve).
- Implemented parallel data processing pipeline with Apache Beam Python SDK and the Dataflow Model to collect 
hundreds of real-time events and power analytics platform.
- Designed MySQL database (application data) and BigQuery database (analytics events).
- Created custom Excel reports via XlsxWriter library (Python) with dozens of reading metrics.

_A learn-to-read app using cognitive science to make reading together fun and impactful._

_Join the Club!_
_We’ve curated a collection of stories for the Junior, Intermediate, and Advanced reading levels. Each of these levels has been carefully researched and analyzed with state-of-the-art metrics._

_Join The Reading Club and get ready to make reading together fun and impactful!_

![GitHub Logo](/Logo-Bee-Reading-001.jpg)


# Example script TextToSpeechModule.cs

```c#
// This is an example file of the source code for The Reading Club, Inc. 
// The Reading Club, Inc. is a private company, so the source doe is private.
// Some links to cloud functions end-points have been taken away for security purposes.

// The intention of this cleaned example script is to showcase the skills and serve as part of the portfolio of htpps://josealvarez97.github.io
// Contact: j.alvarez@minerva.kgi.edu.


using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using WWUtils.Audio;
using ReadingApp.AudioBank;

public class TextToSpeechModule : MonoBehaviour
{
    public AudioBank audioBank;
    public WordVariable selectedWord;


    private const string WORD_TO_AUDIO_URL = "<link to cloud function goes here -- taken away for security purposes>";
    private AudioSource audioSource;
    private Dictionary<string, AudioClip> RuntimeAudioBank { get => audioBank.RuntimeAudioClipBank; }
    private Dictionary<string, byte[]> BytesAudioBank { get => audioBank.BytesAudioBankData.bytesAudioBank;  }



    private void Awake()
    {
        audioSource = this.GetComponent<AudioSource>();
    }

    public void OnWordToAudioDemanded()
    {
        WordToAudio(selectedWord.Word.word);
    }

    public void WordToAudio(string word)
    {
        Debug.Log("WordToAudio Called");
        AudioClip clip;

        if (RuntimeAudioBank.ContainsKey(word))
        {
            Debug.Log("Audio is in Runtime Audio Bank");
            clip = RuntimeAudioBank[word];
            PlayWord(clip);
        }
        else if (BytesAudioBank.ContainsKey(word))
        {
            Debug.Log("Audio is in Bytes Audio Bank");
            var bytes = BytesAudioBank[word];

            clip = WavBytesToAudioClip(bytes, word);
            
            audioBank.AddToRunTimeAudioBank(word, clip);

            PlayWord(clip);
        }
        else
        {
            Debug.Log("Audio is not in any data bank. Audio is to be retrieved from the cloud.");
            StartCoroutine(WordToAudioBODYPARAMETER(word,
                (res) =>
                {
                    Debug.Log("Word to audio result gotten from cloud.");

                    // we get the raw bytes as result/response. we save it to the bytes audio bank
                    audioBank.AddToBytesAudioBank(word, res); // save bytes into persistent memory
                    // we transform it into an audioclip
                    clip = WavBytesToAudioClip(res, word); // get clip
                    // we also save into the runtime audiobank
                    audioBank.AddToRunTimeAudioBank(word, clip); // save clip in runtime for covenience

                    PlayWord(clip);
                },
                (err) =>
                {
                    Debug.LogError("Error in request to cloud of text to audio: " + err);
                }));
        }
    }

    private void PlayWord(AudioClip wordClip)
    {
        this.audioSource.PlayOneShot(wordClip);
    }

    private IEnumerator WordToAudioBODYPARAMETER(string word, System.Action<byte[]> saveClipCallback, System.Action<string> err = null)
    {
        using (var request = new UnityWebRequest(WORD_TO_AUDIO_URL, "POST"))
        {
            // Prepare boy of the request
            WordToAudioReqBody body = new WordToAudioReqBody(word);
            string bodyJSON = JsonUtility.ToJson(body);
            byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(bodyJSON);

            // Attach body and define download handler
            request.uploadHandler = (UploadHandler) new UploadHandlerRaw(bodyRaw);
            request.downloadHandler = (DownloadHandler) new DownloadHandlerBuffer();
            
            // Set request header
            request.SetRequestHeader("Content-Type", "application/json");

            // Send request
            yield return request.SendWebRequest();

            // Handle basic errors
            if (request.isNetworkError || request.isHttpError)
            {
                Debug.Log(request.error);
                if (err != null)
                    err(request.error);

                yield break;
            }

            // Handle a successful request
            Debug.Log("Successful request");
            
            byte[] res = request.downloadHandler.data;

            // save audio raw data
            saveClipCallback(res);
        }
    }


    private IEnumerator WordToAudioReqQUERYPARAMETER(string word, System.Action<AudioClip> saveClipCallBack, System.Action<string> err = null)
    {
        using (var request = UnityWebRequestMultimedia.GetAudioClip(WORD_TO_AUDIO_URL + "?word=" + word, AudioType.WAV))
        {
            yield return request.SendWebRequest();

            if (request.isNetworkError || request.isHttpError)
            {
                Debug.LogError(request.error);          
                if (err != null)
                    err(request.error);
                yield break;
            }
            Debug.Log("Successful request");
            AudioClip resultClip = DownloadHandlerAudioClip.GetContent(request);

            saveClipCallBack(resultClip);
        }
    }
        

    private AudioClip WavBytesToAudioClip(byte[] rawData, string clipName)
    {
        Debug.Log("Converting byte[] to wav AudioClip...");
        WAV wav = new WAV(rawData);
        Debug.Log(wav);
        AudioClip audioClip = AudioClip.Create(clipName, wav.SampleCount, 1, wav.Frequency, false);
        audioClip.SetData(wav.LeftChannel, 0);

        Debug.Log("... successfully converted byte[] to wav AudioClip.");

        return audioClip;
    }



}


public class WordToAudioReqBody
{
    public string word;
    public string auth;

    private const string END_USER_AUTH = "END_USER_AUTH";

    public WordToAudioReqBody(string word)
    {
        this.word = word;
        this.auth = END_USER_AUTH;

    }
}
```
