package ai;

public class CBoard {
    public static int holes = 6;    /* kişi başı 6şar oyuk*/       
    public static int StartingStones;   //oyuk başı 4er taş
    public static int TotalStones; // toplam taş sayısı	
    
    /* THE BOARD:
    							P1[6]  P1[5]  P1[4]  P1[3]  P1[2]  P1[1]
    	                Mancala1                                        Mancala2
    							P2[1]  P2[2]  P2[3]  P2[4]  P2[5]  P2[6]
    indisleme P1[1] den başlayıp, saat yönünün tersi yönde Mancala2 ye doğru gidiyor.*/   
    
    public int[] gameBoard = new int[(holes + 1) * 2]; //14 elemanlı gameBoard dizisi       
    public static int indexP1Home = holes; //indexP1Home=6
    public static int indexP2Home = ((holes + 1) * 2) - 1;//indexP2Home=13   
    /* Statuse göre hamleler
    	(1) sıra P1de
    	(2) sıra P2de
    	(3) P1 kazandı
    	(4) P2 kazandı
    	(5) Berabere*/
    public int Status;
    
    /*yeni oyun başlat*/
    public CBoard() 
    {
        NewGame();
    }
    
    // -----------------------------------------------------------------------------------------
    // Method: NewGame()
    // Değişkenleri: -
    // Dönüş Değeri: -
    // Amaç: değerler atanır 
    //	1. oyuncu için
    // -----------------------------------------------------------------------------------------
    public void NewGame()
    {
        // belirtilen sayıları oyuklara ata
        for (int x = 0; x < holes; x++)
        {
            gameBoard[x] = StartingStones;//P1 oyuklarını doldur
            gameBoard[x + (holes + 1)] = StartingStones;//P2 oyuklarını doldur
        }

        indexP1Home = holes;
        indexP2Home = ((holes + 1) * 2) - 1;

        // kaleleri sıfırla
        gameBoard[indexP1Home] = 0;//gameBoard[6]=0
        gameBoard[indexP2Home] = 0;//gameBoard[13]=0

        // sırayı P1e ver
        Status = 2;
    }
    
    // -----------------------------------------------------------------------------------------
    // Method: ValidMove
    // Değişkenleri: her iki player içinde bin index i 1-6 arasında.
    // Dönüş Değeri: bool: uygun olup olmadığını döndür
    // Amaç: uygun mu değil mi tespit et 
    // -----------------------------------------------------------------------------------------
    public boolean ValidMove(int move)
    {
    	boolean testValid = false;

        if ((move >= 1) && (move <= holes))/*1 <= move <= 6*/
        {
            if ((Status == 1) && (gameBoard[move - 1] > 0))
                testValid = true;/*Sıra P1 de ve ???*/
            if ((Status == 2) && (gameBoard[(move - 1) + (holes + 1)] > 0))
                testValid = true;/*Sıra P2 de ve ???*/
        }
        return testValid;
    }
    
 // -----------------------------------------------------------------------------------------
    // Method: MakeMove
    // Değişkenleri: oynanmak istenen hamle
    // Dönüş Değeri: -
    // Amaç: Taşları saatin tersi yönünde dağıt
     // -----------------------------------------------------------------------------------------
    public void MakeMove(int move)
    {
        int numStonesInBin;
        int tempCounter = 0;

        int arrayIndexOfP1Move = move - 1;
        int indexOfP1FirstDropStone = arrayIndexOfP1Move + 1;

        int arrayIndexOfP2Move = (move - 1) + (holes + 1);
        int indexOfP2FirstDropStone = arrayIndexOfP2Move + 1;

        int moduloIndex = (holes + 1) * 2;

        int indexOfP1FirstBin = 0;
        int indexOfP1LastBin = indexP1Home - 1;

        int indexOfP2FirstBin = holes + 1;
        int indexOfP2LastBin = indexP2Home - 1;

        int x;//counter
        boolean GoAgain = false;
        int sum1, sum2;
        boolean GameOver = false;

        // hamle uygun ise oyuncu bu hamleyi yapabilir
        if (ValidMove(move))
        {
            switch (Status)
            {
                case 1:
                    {
                        // Oyuncu tarafındaki taşlar
                        numStonesInBin = gameBoard[arrayIndexOfP1Move];

                        // Boş oyuklar kaldırılır
                        gameBoard[arrayIndexOfP1Move] = 0;

                        // Taşları saatin tersi yönünde dağıt
                        // Rakibin kalesini atla
                        for (x = 0; x < (numStonesInBin + tempCounter); x++)
                        {
                            // karşının kalesi değilse taş koy
                            if (!(((indexOfP1FirstDropStone + x) % moduloIndex) == indexP2Home))
                            {
                                gameBoard[(indexOfP1FirstDropStone + x) % moduloIndex]++;
                            }
                            else 
                                tempCounter++;
                        }

                        // capture u kontrol et
                        int indexOfBinOfLastStone = (indexOfP1FirstDropStone + x - 1) % (moduloIndex);

                        // bizim tarafta isek
                        if ((indexOfBinOfLastStone >= indexOfP1FirstBin) &&
                                (indexOfBinOfLastStone <= indexOfP1LastBin))
                        {
                            // goal ise
                            int currBin = indexOfBinOfLastStone + 1;//currBin 1-6 arası bin no su.	

                            // son taşın atıldığı yer
                            int oppositeBin = holes + 1 - currBin;//currBin in karşısındaki bin no su.

                            int indexOfOppositeBin = oppositeBin + holes;

                            if ((gameBoard[indexOfBinOfLastStone] == 1) &&//son taşın koyulduğu bin de 1 taş olmalı
                                    (gameBoard[indexOfOppositeBin] > 0) &&//son taşın koyulduğu binin karşısı boş olmamalı
                                    (indexOfBinOfLastStone != indexP1Home))//son taşın koyulduğu home olmamalı.
                            {
                                gameBoard[indexP1Home] += gameBoard[indexOfBinOfLastStone] +
                                                          gameBoard[indexOfOppositeBin];
                                gameBoard[indexOfBinOfLastStone] = 0;
                                gameBoard[indexOfOppositeBin] = 0;
                            }
                        }

                        if (indexOfBinOfLastStone == indexP1Home)
                            GoAgain = true;
                        break;
                    }

                // P2 oyuncunun hamlesi ise, manipule et
                case 2:
                    {
                        // p2 tarafındaki taşların sayısı
                        numStonesInBin = gameBoard[arrayIndexOfP2Move];

                        // Boş oyuklar kaldırılır
                        gameBoard[arrayIndexOfP2Move] = 0;

                        // Taşları saatin tersi yönünde dağıt
                        // Rakibin kalesini atla
                        for (x = 0; x < (numStonesInBin + tempCounter); x++)
                        {
                            // kale değilse taş koy
                            if (!(((indexOfP2FirstDropStone + x) % moduloIndex) == indexP1Home))
                            {
                                gameBoard[(indexOfP2FirstDropStone + x) % moduloIndex]++;
                            }
                            else
                                tempCounter++; 
                        }

                        //  capture u kontrol et
                        int indexOfBinOfLastStone = (indexOfP2FirstDropStone + x - 1) % moduloIndex;

                        // karşı tarafta isek
                        if ((indexOfBinOfLastStone >= indexOfP2FirstBin) && (indexOfBinOfLastStone <= indexOfP2LastBin))
                        {
                            // son taşın bırakıldığı oyuk
                            int currBin = indexOfBinOfLastStone - holes;

                            int oppositeBin = holes + 1 - currBin;

                            int indexOfOppositeBin = oppositeBin - 1;

                            if ((gameBoard[indexOfBinOfLastStone] == 1) && (gameBoard[indexOfOppositeBin] > 0) && (indexOfBinOfLastStone != indexP2Home))
                            {
                                gameBoard[indexP2Home] += gameBoard[indexOfBinOfLastStone] + gameBoard[indexOfOppositeBin];
                                gameBoard[indexOfBinOfLastStone] = 0;
                                gameBoard[indexOfOppositeBin] = 0;
                            }
                        }

                        if (indexOfBinOfLastStone == indexP2Home)
                            GoAgain = true;
                        break;
                    }
                default: break;
            }

            // kimin oynayacağını belirle
            if (!GoAgain)
            {
                // P1 oynadı P2ye geç
                if (Status == 1)
                    Status = 2;
                // P2 oynadı P1e geç
                else if (Status == 2)
                    Status = 1;
            }

            sum1 = sum2 = 0;

            for (x = 0; x < holes; x++)
            {
                sum1 += gameBoard[x];
                sum2 += gameBoard[x + (holes + 1)];
            }

            //P1 tarafında taş kalmadıysa kalan taşları aktar oyunu bitir
            if (sum1 == 0)
            {
                // P2 tarafında kalan taşları aktar
                gameBoard[indexP2Home] += sum2;

                // oyukarı sıfırla
                for (x = 0; x < holes; x++)
                     gameBoard[x + (holes + 1)] = 0;

                // oyun bittiğini belirt
                GameOver = true;
            }

            // P2 tarafında taş kalmadıysa kalan taşları aktar oyunu bitir
            else if (sum2 == 0)
            {
                // P1 tarafında kalan taşları aktar
                gameBoard[indexP1Home] += sum1;

                // oyukarı sıfırla
                for (x = 0; x < holes; x++)
                    gameBoard[x] = 0;

                // oyun bittiğini belirt
                GameOver = true;
            }

            // Kim kazandı?
            if (GameOver)
            {
                // P1's
                if (gameBoard[indexP1Home] > gameBoard[indexP2Home])
                    Status = 3;
                // P2's 
                else if (gameBoard[indexP2Home] > gameBoard[indexP1Home])
                    Status = 4;
                // Berabere
                else
                    Status = 5;
            }
        }
    }
    
    public void OutputBoard(int bin){}			// ekrana yazdırma
    
    // -----------------------------------------------------------------------------------------
    // Method: MakeCopyIn
    // Değişkenleri: CBoard *destination (pointer to the copy of the current board's configuration)
    // Dönüş Değeri: -
    // Amaç: kullanılan tahtanın ayarlarını kopyalar
    // -----------------------------------------------------------------------------------------
    public void MakeCopyIn(CBoard destination)
    {
        //destination = new CBoard();
        // dizideki değerleri kopyala
        for (int x = 0; x < ((holes + 1) * 2); x++)
            destination.gameBoard[x] = this.gameBoard[x];
   
        // status değerini kopyala
        destination.Status = Status;
        CBoard.indexP1Home = holes;
        CBoard.indexP2Home = ((holes + 1) * 2) - 1;
    }
    
    // -------------------------------------------------------------------
    // struct: Node
    // Amaç: ağaç yapısındaki bilgileri tutmak için
    // -------------------------------------------------------------------
//    class Node
//    {
//        public CBoard board;
//        public int evalFunc;
//        public int minimaxMove;
//        public int nodeDepth;
//    }; 
}


