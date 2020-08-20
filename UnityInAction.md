## C# keypoint
##### C# EventSystem
```C#
class MainClass{
    static void Main(){new Publisher().TriggleEvent();}
}
class CustomEventArgs : EventArgs {}
class Publisher
{
    public event EventHandler<CustomEventArgs> EventSubscriber;
    public void Register(EventHandler<CustomEventArgs> handler){EventSubscriber += handler;}
    public void TriggleEvent(){EventSubscriber(this, new CustomEventArgs());}
}
class Parent{
    public Publisher m_Subscriber;
    public Parent(){m_Subscriber = new Publisher();}
}
class Subscriber:Parent{
    public Subscriber(){m_Subscriber.Register(OnHandleEvent);}
    public void OnHandleEvent(object source, CustomEventArgs args) {}
}
```
##### C# Messenger
```C#
using System;
using System.Collections.Generic;
using System.Linq;
 
public enum MessengerMode {
    DONT_REQUIRE_LISTENER,
    REQUIRE_LISTENER,
}
 
static internal class MessengerInternal {
    readonly public static Dictionary<string, Delegate> eventTable = new Dictionary<string, Delegate>();
    static public MessengerMode DEFAULT_MODE = MessengerMode.REQUIRE_LISTENER;
 
    static public void AddListener(string eventType, Delegate callback) {
        MessengerInternal.OnListenerAdding(eventType, callback);
        eventTable[eventType] = Delegate.Combine(eventTable[eventType], callback);
    }
 
    static public void RemoveListener(string eventType, Delegate handler) {
        MessengerInternal.OnListenerRemoving(eventType, handler);   
        eventTable[eventType] = Delegate.Remove(eventTable[eventType], handler);
        MessengerInternal.OnListenerRemoved(eventType);
    }
 
    static public T[] GetInvocationList<T>(string eventType) {
        Delegate d;
        if(eventTable.TryGetValue(eventType, out d)) {
            try {
                return d.GetInvocationList().Cast<T>().ToArray();
            } catch {
                throw MessengerInternal.CreateBroadcastSignatureException(eventType);
            }
        }
        return new T[0];
    }
 
    static public void OnListenerAdding(string eventType, Delegate listenerBeingAdded) {
        if (!eventTable.ContainsKey(eventType)) {
            eventTable.Add(eventType, null);
        }
 
        var d = eventTable[eventType];
        if (d != null && d.GetType() != listenerBeingAdded.GetType()) {
            throw new ListenerException(string.Format("Attempting to add listener with inconsistent signature for event type {0}. Current listeners have type {1} and listener being added has type {2}", eventType, d.GetType().Name, listenerBeingAdded.GetType().Name));
        }
    }
 
    static public void OnListenerRemoving(string eventType, Delegate listenerBeingRemoved) {
        if (eventTable.ContainsKey(eventType)) {
            var d = eventTable[eventType];
 
            if (d == null) {
                throw new ListenerException(string.Format("Attempting to remove listener with for event type {0} but current listener is null.", eventType));
            } else if (d.GetType() != listenerBeingRemoved.GetType()) {
                throw new ListenerException(string.Format("Attempting to remove listener with inconsistent signature for event type {0}. Current listeners have type {1} and listener being removed has type {2}", eventType, d.GetType().Name, listenerBeingRemoved.GetType().Name));
            }
        } else {
            throw new ListenerException(string.Format("Attempting to remove listener for type {0} but Messenger doesn't know about this event type.", eventType));
        }
    }
 
    static public void OnListenerRemoved(string eventType) {
        if (eventTable[eventType] == null) {
            eventTable.Remove(eventType);
        }
    }
 
    static public void OnBroadcasting(string eventType, MessengerMode mode) {
        if (mode == MessengerMode.REQUIRE_LISTENER && !eventTable.ContainsKey(eventType)) {
            throw new MessengerInternal.BroadcastException(string.Format("Broadcasting message {0} but no listener found.", eventType));
        }
    }
 
    static public BroadcastException CreateBroadcastSignatureException(string eventType) {
        return new BroadcastException(string.Format("Broadcasting message {0} but listeners have a different signature than the broadcaster.", eventType));
    }
 
    public class BroadcastException : Exception {
        public BroadcastException(string msg)
            : base(msg) {
        }
    }
 
    public class ListenerException : Exception {
        public ListenerException(string msg)
            : base(msg) {
        }
    }
}
 
// No parameters
static public class Messenger { 
    static public void AddListener(string eventType, Action handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void AddListener<TReturn>(string eventType, Func<TReturn> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void RemoveListener(string eventType, Action handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void RemoveListener<TReturn>(string eventType, Func<TReturn> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void Broadcast(string eventType) {
        Broadcast(eventType, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast<TReturn>(string eventType, Action<TReturn> returnCall) {
        Broadcast(eventType, returnCall, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast(string eventType, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Action>(eventType);
 
        foreach(var callback in invocationList)
            callback.Invoke();
    }
 
    static public void Broadcast<TReturn>(string eventType, Action<TReturn> returnCall, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Func<TReturn>>(eventType);
 
        foreach(var result in invocationList.Select(del => del.Invoke()).Cast<TReturn>()) {
            returnCall.Invoke(result);
        }
    }
}
 
// One parameter
static public class Messenger<T> {
    static public void AddListener(string eventType, Action<T> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void AddListener<TReturn>(string eventType, Func<T, TReturn> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void RemoveListener(string eventType, Action<T> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void RemoveListener<TReturn>(string eventType, Func<T, TReturn> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void Broadcast(string eventType, T arg1) {
        Broadcast(eventType, arg1, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, Action<TReturn> returnCall) {
        Broadcast(eventType, arg1, returnCall, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast(string eventType, T arg1, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Action<T>>(eventType);
 
        foreach(var callback in invocationList)
            callback.Invoke(arg1);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, Action<TReturn> returnCall, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Func<T, TReturn>>(eventType);
 
        foreach(var result in invocationList.Select(del => del.Invoke(arg1)).Cast<TReturn>()) {
            returnCall.Invoke(result);
        }
    }
}
 
 
// Two parameters
static public class Messenger<T, U> { 
    static public void AddListener(string eventType, Action<T, U> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void AddListener<TReturn>(string eventType, Func<T, U, TReturn> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void RemoveListener(string eventType, Action<T, U> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void RemoveListener<TReturn>(string eventType, Func<T, U, TReturn> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void Broadcast(string eventType, T arg1, U arg2) {
        Broadcast(eventType, arg1, arg2, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, U arg2, Action<TReturn> returnCall) {
        Broadcast(eventType, arg1, arg2, returnCall, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast(string eventType, T arg1, U arg2, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Action<T, U>>(eventType);
 
        foreach(var callback in invocationList)
            callback.Invoke(arg1, arg2);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, U arg2, Action<TReturn> returnCall, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Func<T, U, TReturn>>(eventType);
 
        foreach(var result in invocationList.Select(del => del.Invoke(arg1, arg2)).Cast<TReturn>()) {
            returnCall.Invoke(result);
        }
    }
}
 
 
// Three parameters
static public class Messenger<T, U, V> { 
    static public void AddListener(string eventType, Action<T, U, V> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void AddListener<TReturn>(string eventType, Func<T, U, V, TReturn> handler) {
        MessengerInternal.AddListener(eventType, handler);
    }
 
    static public void RemoveListener(string eventType, Action<T, U, V> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void RemoveListener<TReturn>(string eventType, Func<T, U, V, TReturn> handler) {
        MessengerInternal.RemoveListener(eventType, handler);
    }
 
    static public void Broadcast(string eventType, T arg1, U arg2, V arg3) {
        Broadcast(eventType, arg1, arg2, arg3, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, U arg2, V arg3, Action<TReturn> returnCall) {
        Broadcast(eventType, arg1, arg2, arg3, returnCall, MessengerInternal.DEFAULT_MODE);
    }
 
    static public void Broadcast(string eventType, T arg1, U arg2, V arg3, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Action<T, U, V>>(eventType);
 
        foreach(var callback in invocationList)
            callback.Invoke(arg1, arg2, arg3);
    }
 
    static public void Broadcast<TReturn>(string eventType, T arg1, U arg2, V arg3, Action<TReturn> returnCall, MessengerMode mode) {
        MessengerInternal.OnBroadcasting(eventType, mode);
        var invocationList = MessengerInternal.GetInvocationList<Func<T, U, V, TReturn>>(eventType);
 
        foreach(var result in invocationList.Select(del => del.Invoke(arg1, arg2, arg3)).Cast<TReturn>()) {
            returnCall.Invoke(result);
        }
    }
}
```
#### Messenger 精简版
```C#
static internal class MessageQueueInternal {
    private static Dictionary<string, Delegate> m_Subscriber = new Dictionary<string, Delegate>();
    public static void AddListener(string eventType, Delegate handler) {
        if (!m_Subscriber.ContainsKey(eventType)) {
            m_Subscriber.Add(eventType, null);
        }
        m_Subscriber[eventType] = Delegate.Combine(m_Subscriber[eventType],handler);
    }
    public static Delegate[] GetInvokeList(string eventType){
        Delegate delegateList;
        if (m_Subscriber.TryGetValue(eventType,out delegateList)) {
            return delegateList.GetInvocationList();
        }
        return new Delegate[0];
    }
}
static public class MessageQueue<T>{
    public static void AddListener(string eventType,Action<T> handler) {
        MessageQueueInternal.AddListener(eventType,handler);
    }
    public static void Broadcast(string eventType, T arg) {
        var invokeList = MessageQueueInternal.GetInvokeList(eventType);
        foreach (var call in invokeList) {
            call.DynamicInvoke(arg);
        }
    }
}
```
### Custom Playable graph
##### Custom track
```C#
[TrackColor(1f,0f,0f)]
[TrackClipType(typeof(CustomPlayableAssets))]
[TrackBindingType(typeof(CustomTrack))]
public class CustomTrack : TrackAsset
{
    protected override Playable CreatePlayable(PlayableGraph graph, GameObject gameObject, TimelineClip clip)
    {
        return base.CreatePlayable(graph, gameObject, clip);
    }

    public override Playable CreateTrackMixer(PlayableGraph graph, GameObject go, int inputCount)
    {
        return base.CreateTrackMixer(graph, go, inputCount);
    }
}
```
##### Custom animation mixer
```C#
public class CustomPlayMixer : MonoBehaviour
{
    public AnimationClip AnimationClip1;
    public AnimationClip AnimationClip2;
    public float weight;
    private PlayableGraph m_PlayableGraph;
    private AnimationMixerPlayable m_AnimationMixerPlayable;
    private void Start()
    {
        m_AnimationMixerPlayable =
            AnimationPlayableUtilities.PlayMixer(GetComponent<Animator>(), 2, out m_PlayableGraph);
        AnimationClipPlayable clipPlayable1 = AnimationClipPlayable.Create(m_PlayableGraph,AnimationClip1);
        AnimationClipPlayable clipPlayable2 = AnimationClipPlayable.Create(m_PlayableGraph,AnimationClip2);
        m_PlayableGraph.Connect(clipPlayable1, 0, m_AnimationMixerPlayable, 0);
        m_PlayableGraph.Connect(clipPlayable2, 0, m_AnimationMixerPlayable, 1);
        AnimationPlayableOutput output = AnimationPlayableOutput.Create(m_PlayableGraph,"AnimationOutput",GetComponent<Animator>());
        output.SetSourcePlayable(m_AnimationMixerPlayable);
        m_AnimationMixerPlayable.Play();
    }
    private void Update()
    {
        weight = Mathf.Clamp01(weight);
        m_AnimationMixerPlayable.SetInputWeight(0,1-weight);
        m_AnimationMixerPlayable.SetInputWeight(1,weight);
    }
    private void OnDestroy()
    {
        m_PlayableGraph.Destroy();
    }
}
```
##### Custom audio mixer
```C#
//音频混合（需要audio sourcce）
public class AudioMixer : MonoBehaviour
{
    public AudioClip m_AudioClip1;
    public AudioClip m_AudioClip2;
    public float weight;
    private PlayableGraph m_PlayableGraph;
    public AudioMixerPlayable m_AudioMixerPlayable;
    private void Start()
    {
        m_PlayableGraph = PlayableGraph.Create();
        m_AudioMixerPlayable = AudioMixerPlayable.Create(m_PlayableGraph,2);
        var audioClipPlayable1 = AudioClipPlayable.Create(m_PlayableGraph, m_AudioClip1, true);
        var audioClipPlayable2 = AudioClipPlayable.Create(m_PlayableGraph, m_AudioClip2, true);
        m_PlayableGraph.Connect(audioClipPlayable1,0,m_AudioMixerPlayable,0);
        m_PlayableGraph.Connect(audioClipPlayable2,0,m_AudioMixerPlayable,1);
        var audioPlayableOutput = AudioPlayableOutput.Create(m_PlayableGraph, "Audio", GetComponent<AudioSource>());
        audioPlayableOutput.SetSourcePlayable(m_AudioMixerPlayable);
        m_AudioMixerPlayable.Play();
    }
    private void Update()
    {
        weight = Mathf.Clamp01(weight);
        m_AudioMixerPlayable.SetInputWeight(0,1-weight);
        m_AudioMixerPlayable.SetInputWeight(1,weight);
    }

    private void OnDisable()
    {
        m_PlayableGraph.Destroy();
    }
}
```