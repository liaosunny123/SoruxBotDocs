```csharp

//plugin:
public void PluginAction(){
string message = await LongCommunicate.Read();

}

//LongCommunicate
public async Task<string> LongCommunicate(string message){
if(message...){
//如果符合要求那么就返回
}else{
//如果不符合要求就继续等待
}
}



```