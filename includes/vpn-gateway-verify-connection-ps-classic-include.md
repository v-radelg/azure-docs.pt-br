Você pode usar o `Get-AzureVNetConnection` para verificar a conexão com um gateway de rede virtual clássica. 

1. Use o seguinte exemplo de cmdlet, configurando os valores para coincidirem com os seus. O nome da rede virtual deve ficar entre aspas se contiver espaços.
   
        Get-AzureVNetConnection "Group ClassicRG ClassicVNet"
2. Após o cmdlet ter sido concluído, exiba os valores. No exemplo abaixo, o Estado de Conectividade é exibido como ‘Conectado’ e é possível ver os bytes de entrada e saída.

        ConnectivityState         : Connected
        EgressBytesTransferred    : 181664
        IngressBytesTransferred   : 182080
        LastConnectionEstablished : 1/7/2016 12:40:54 AM
        LastEventID               : 24401
        LastEventMessage          : The connectivity state for the local network site 'RMVNetLocal' changed from Connecting to
                                    Connected.
        LastEventTimeStamp        : 1/7/2016 12:40:54 AM
        LocalNetworkSiteName      : RMVNetLocal


<!--HONumber=Jan17_HO3-->


