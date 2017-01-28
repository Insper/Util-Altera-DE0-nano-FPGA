library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;
use IEEE.std_logic_unsigned.all;
use work.all;

entity sdramTeste is

    PORT(
        -- Sistema
        CLK_50  : IN  STD_LOGIC; -- 50 Mhz
  
        -- I/Os digital
        LED : out std_logic_vector(7 downto 0);
        KEY : in  std_logic_vector(1 downto 0);

        -- SDRAM
        DRAM_ADDR   : out   std_logic_vector(12 downto 0);
        DRAM_BA     : out   std_logic_vector(1  downto 0);
        DRAM_CAS_N  : out   std_logic;  
        DRAM_CKE    : out   std_logic;
        DRAM_CLK    : out   std_logic;
        DRAM_CS_N   : out   std_logic;
        DRAM_DQ     : inout std_logic_vector(15 downto 0);
        DRAM_DQM    : out   std_logic_vector(01 downto 0);
        DRAM_RAS_N  : out   std_logic;
        DRAM_WE_N   : out   std_logic
    );
end entity;

ARCHITECTURE logic OF sdramTeste IS

    component sdram is
        port (
            sdram_wire_addr        : out   std_logic_vector(12 downto 0);                    -- addr
            sdram_wire_ba          : out   std_logic_vector(1 downto 0);                     -- ba
            sdram_wire_cas_n       : out   std_logic;                                        -- cas_n
            sdram_wire_cke         : out   std_logic;                                        -- cke
            sdram_wire_cs_n        : out   std_logic;                                        -- cs_n
            sdram_wire_dq          : inout std_logic_vector(15 downto 0) := (others => 'X'); -- dq
            sdram_wire_dqm         : out   std_logic_vector(1 downto 0);                     -- dqm
            sdram_wire_ras_n       : out   std_logic;                                        -- ras_n
            sdram_wire_we_n        : out   std_logic;                                        -- we_n
            sdram_s1_address       : in    std_logic_vector(23 downto 0) := (others => 'X'); -- address
            sdram_s1_byteenable_n  : in    std_logic_vector(1 downto 0)  := (others => 'X'); -- byteenable_n
            sdram_s1_chipselect    : in    std_logic                     := 'X';             -- chipselect
            sdram_s1_writedata     : in    std_logic_vector(15 downto 0) := (others => 'X'); -- writedata
            sdram_s1_read_n        : in    std_logic                     := 'X';             -- read_n
            sdram_s1_write_n       : in    std_logic                     := 'X';             -- write_n
            sdram_s1_readdata      : out   std_logic_vector(15 downto 0);                    -- readdata
            sdram_s1_readdatavalid : out   std_logic;                                        -- readdatavalid
            sdram_s1_waitrequest   : out   std_logic;                                        -- waitrequest
            clk_clk                : in    std_logic                     := 'X';             -- clk
            reset_reset_n          : in    std_logic                     := 'X'              -- reset_n
        );
    end component sdram;

    signal sdram_s1_address       :  std_logic_vector(23 downto 0); 
    signal sdram_s1_byteenable_n  :  std_logic_vector(1  downto 0) ; 
    signal sdram_s1_chipselect    :  std_logic                    ; 
    signal sdram_s1_writedata     :  std_logic_vector(15 downto 0); 
    signal sdram_s1_read_n        :  std_logic                    ; 
    signal sdram_s1_write_n       :  std_logic                    ; 
    signal sdram_s1_readdata      :  std_logic_vector(15 downto 0);
    signal sdram_s1_readdatavalid :  std_logic                    ; 
    signal sdram_s1_waitrequest   :  std_logic                    ; 

    signal address                :  integer range 0 to 4095 := 50;
    signal dataWrite              :  unsigned(7 downto 0)    := x"55";

    type   STATE_TYPE IS (s0, sW1, sW2, sR0, sR1, sR2, sR3);
    signal state   : STATE_TYPE := s0;

    
    signal RST     : STD_LOGIC := '1';    

begin

    u0 : component sdram
        port map (
            sdram_wire_addr        => DRAM_ADDR,
            sdram_wire_ba          => DRAM_BA,
            sdram_wire_cas_n       => DRAM_CAS_N,
            sdram_wire_cke         => DRAM_CKE,
            sdram_wire_cs_n        => DRAM_CS_N,
            sdram_wire_dq          => DRAM_DQ,
            sdram_wire_dqm         => DRAM_DQM,
            sdram_wire_ras_n       => DRAM_RAS_N,
            sdram_wire_we_n        => DRAM_WE_N,
            sdram_s1_address       => sdram_s1_address     , 
            sdram_s1_byteenable_n  => sdram_s1_byteenable_n, 
            sdram_s1_chipselect    => sdram_s1_chipselect  , 
            sdram_s1_writedata     => sdram_s1_writedata   , 
            sdram_s1_read_n        => sdram_s1_read_n      , 
            sdram_s1_write_n       => sdram_s1_write_n     , 
            sdram_s1_readdata      => sdram_s1_readdata    , 
            sdram_s1_readdatavalid => sdram_s1_readdatavalid,
            sdram_s1_waitrequest   => sdram_s1_waitrequest , 
            clk_clk                => CLK_50,
            reset_reset_n          => rst
        );


    -- Botao evento (incrementa contador para escrita)
    -- KEY 1 : incrementa e salva contador na memoria
    -- KEY 0 : decrementa e lê valor da memoria
    process(CLK_50)
    begin
        if(rising_edge(CLK_50)) then
            case state is

                -----------------------
                -- Default state		
                -----------------------
                when s0 =>
                    sdram_s1_chipselect <= '0';
                    sdram_s1_write_n    <= '1';  
                    sdram_s1_read_n     <= '1';  

                    if(key(1) = '0') then -- save
                        sdram_s1_chipselect <= '1';
                        address             <= address;
                        state               <= sW1;
                    else
                        state               <= s0;
                    end if;

                    if(key(0) = '0') then -- read
                        sdram_s1_chipselect <= '1';
                        address             <= address - 1;
                        state               <= sR1;
                    else
                        state               <= s0;
                    end if;

                -----------------------
                -- Write states
                -----------------------
                when sW1 =>
                    --sdram_s1_address    <= std_logic_vector(to_unsigned(address,24)); 
                    if(key(1) = '1') then -- released
                        sdram_s1_write_n <= '0';
                        state            <= sW2;
                    else
                        state            <= sW1;
                    end if;

                when sW2 =>
                    sdram_s1_write_n    <= '1';
                    address             <= address + 1;
                    state               <= s0;
                
                ----------------------
                -- Read states
                ----------------------
                when sR1 =>
                    --sdram_s1_address    <= std_logic_vector(to_unsigned(address,24)); 
                    if(key(0) = '1') then -- released
                        sdram_s1_read_n <= '0';
                        state           <= sR2;
                    else
                        state           <= sR1;
                    end if;

                when sR2 =>

                    if( sdram_s1_readdatavalid = '1') then
                        sdram_s1_read_n <= '1';
                        LED             <= sdram_s1_readdata(7 downto 0);
                        state           <= s0;
                    else
                        state           <= sR2;
                    end if;

                when others =>
                    state <= s0;

            end case;

        end if; 
    end process;

                sdram_s1_address    <= std_logic_vector(to_unsigned(address,24)); 
  sdram_s1_writedata(7 downto 0)    <= std_logic_vector(dataWrite);
  sdram_s1_writedata(15 downto 8)   <= (others => '0');
  sdram_s1_byteenable_n 				<= (others => '0');
  sdram_s1_waitrequest  				<= '0'; 


end;


