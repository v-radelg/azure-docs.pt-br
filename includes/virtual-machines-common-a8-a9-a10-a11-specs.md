
## <a name="key-features"></a>Principais recursos
* **Hardware de alto desempenho** – Estas instâncias foram projetadas e otimizadas para aplicativos de computação e rede intensivas, incluindo aplicativos HPC (computação de alto desempenho) e de lote, modelagem e simulações em larga escala. 
  
    Detalhes sobre o processador do Intel Xeon E5-2667 v3 (usado na série H) e o processador Intel Xeon E5-2670 (em A8 a A11), incluindo extensões do conjunto de instruções com suporte, estão no site Intel.com. 
* **Projetado para clusters HPC** – Implante várias instâncias de computação intensiva no Azure para criar um cluster HPC autônomo ou adicionar capacidade a um cluster local. Se desejar, implante ferramentas de agendamento de trabalho e gerenciamento de cluster. Ou então, use as instâncias de trabalho de computação intensiva em outro serviço do Azure como Lote do Azure.
* **Conexão de rede RDMA para aplicativos MPI** – Um subconjunto das instâncias de computação intensiva (H16r, H16mr, A8 e A9) apresenta um segundo adaptador de rede para conectividade RDMA (acesso remoto direto à memória). Essa interface é além do adaptador de rede do Azure padrão disponível para outros tamanhos de VM. 
  
    Essa interface permite que instâncias compatíveis com RDMA se comuniquem entre si em uma rede InfiniBand, operando em taxas FDR para máquinas virtuais H16r e H16mr e taxas QDR para máquinas virtuais A8 e A9. Os recursos RDMA expostos nessas máquinas virtuais podem melhorar a escalabilidade e o desempenho de determinados aplicativos MPI (Interface de Transmissão de Mensagens) do Linux e do Windows. Veja [Acesso à rede RDMA](#access-to-the-rdma-network) neste artigo para obter os requisitos.

## <a name="deployment-considerations"></a>Considerações de implantação
* **Assinatura do Azure** – Se desejar implantar mais do que um pequeno número de instâncias de computação intensiva, considere uma assinatura pré-paga ou outras opções de compra. Se estiver usando uma [conta gratuita do Azure](https://azure.microsoft.com/free/), você poderá usar apenas um número limitado de núcleos de computação do Azure.
* **Preço e disponibilidade** – Os tamanhos de VM de computação intensiva são oferecidos apenas no tipo de preço Standard. Confira [Produtos disponíveis por região](https://azure.microsoft.com/regions/services/) para ver a disponibilidade nas regiões do Azure. 
* **Cota de núcleos** – Talvez você precise aumentar a cota de núcleos em sua assinatura do Azure do padrão de 20 núcleos por assinatura (se usar o modelo de implantação clássica) ou 20 núcleos por região (se usar o modelo de implantação do Gerenciador de Recursos). Sua assinatura também pode limitar o número de núcleos que você pode implantar em determinadas famílias de tamanho de VM, incluindo a série de H. Para solicitar um aumento de cota, [abra uma solicitação de atendimento ao cliente online](../articles/azure-supportability/how-to-create-azure-support-request.md) gratuitamente. (Os limites padrão podem variar dependendo de sua categoria de assinatura.)
  
  > [!NOTE]
  > Entre em contato com o Suporte do Azure se precisar de capacidade em larga escala. Cotas do Azure são limites de crédito, não garantias de capacidade. Independentemente de sua cota, você é cobrado apenas pelo núcleos utilizados.
  > 
  > 
* **Rede virtual** – Não é necessário ter uma [rede virtual](https://azure.microsoft.com/documentation/services/virtual-network/) do Azure para usar instâncias de computação intensiva. No entanto, talvez você precise ter, pelo menos, uma rede virtual do Azure baseada em nuvem para vários cenários de implantação ou uma conexão site a site se precisar acessar recursos locais, como um servidor de licença de aplicativo. Se for necessário, você precisará criar uma nova rede virtual para implantar as instâncias. Não há suporte para a adição de VMs de computação intensiva a uma rede virtual em um grupo de afinidades.
* **Serviço de nuvem ou conjunto de disponibilidade** – Para usar a rede RDMA do Azure, implante VMs compatíveis com RDMA no mesmo serviço de nuvem (se usar o modelo de implantação clássica) ou no mesmo conjunto de disponibilidade (se usar o modelo de implantação do Azure Resource Manager). Se você usar o Lote do Azure, as VMs compatíveis com RDMA deverão estar no mesmo pool.
* **Redimensionamento** – Devido ao hardware especializado usado nas instâncias de computação intensiva, você só pode redimensionar instâncias de computação intensiva dentro da mesma família de tamanho (série H ou série A de computação intensiva). Por exemplo, somente é possível redimensionar uma VM da série H de um tamanho da série H para outro. Além disso, não há suporte para o redimensionamento de um tamanho sem computação intensiva para um tamanho de computação intensiva.  
* **Espaço de endereço de rede RDMA** - A rede RDMA no Azure reserva o espaço de endereço 172.16.0.0/16. Para executar aplicativos MPI em instâncias implantadas em uma rede virtual do Azure, verifique se o espaço do endereço de rede virtual não se sobrepõe à rede RDMA.



<!--HONumber=Nov16_HO4-->


