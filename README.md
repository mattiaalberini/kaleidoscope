## Grammatica primo livello 

Aggiunte le ***variabili globali***. Ho dovuto modificare anche la classe ***VariableExprAST***

```cpp
class GlobalVariableAST: public RootAST{
  private:
    std::string Name;

  public:
    GlobalVariableAST(const std::string Name);
    Value* codegen(driver& drv) override;
    const std::string& getName();
};
```

```cpp
Value *VariableExprAST::codegen(driver& drv) {
  AllocaInst *A = drv.NamedValues[Name];
  if (!A) {
    GlobalVariable *gv = module->getNamedGlobal(Name);
    if(!gv){
      return LogErrorV("Variabile "+Name+" non definita");}
    else{
      return builder->CreateLoad(gv->getValueType(), gv, Name.c_str());}
  }
  return builder->CreateLoad(A->getAllocatedType(), A, Name.c_str());
}
```


Aggiunta la classe virtuale ***StmtAST*** che rappresenta gli statements

```cpp
class StmtAST : public RootAST {};
```


Definita la classe virtuale ***BlockAST*** che rappresenta un blocco di codice

```cpp
class BlockAST : public ExprAST {
  private:
    std::vector<VarBindingAST*> Def;
    std::vector<StmtAST*> Stmts;
  public:
  BlockAST(std::vector<VarBindingAST*> Def, std::vector<StmtAST*> Stmts);
  BlockAST(std::vector<StmtAST*> Stmts);
  Value *codegen(driver& drv) override;
};
```


La classe ***VarBindingAST*** si occupa sia della definizione sia dell'assegnamento

```cpp
class VarBindingAST: public StmtAST {
  private:
    const std::string Name;
    ExprAST* Val;
  public:
    VarBindingAST(const std::string Name, ExprAST* Val);
    Value *codegen(driver& drv) override;
    const std::string& getName() const;
};
```


## Grammatica secondo livello 

Aggiunti gli statements ***if*** e ***for***.

```cpp
class IfStmtAST : public StmtAST {
  private:
    ExprAST* Cond;
    StmtAST* TrueExp;
    StmtAST* FalseExp;
  public:
    IfStmtAST(ExprAST* Cond, StmtAST* TrueExp, StmtAST* FalseExp);
    Value *codegen(driver& drv) override;
};
```

```cpp
class ForStmtAST : public StmtAST {
  private: 
    VarBindingAST* Init;
    ExprAST* Cond;
    VarBindingAST* Step;
    StmtAST* Body;

  public:
    ForStmtAST(VarBindingAST* Init, ExprAST* Cond, VarBindingAST* Step, StmtAST* Body);
    Value *codegen(driver& drv) override;
};
```


## Grammatica terzo livello

Aggiunti gli operatori di algebra booleana ***AND***, ***OR*** e ***NOT***.

```cpp
  Value *BinaryExprAST::codegen(driver& drv) {
    if(Op == 'n') {
      //Not
      Value *L = LHS->codegen(drv);
      if (!L) return nullptr;
      else return builder->CreateNot(L,"notres");
    }
    Value *L = LHS->codegen(drv);
    Value *R = RHS->codegen(drv);
    if (!L || !R) 
      return nullptr;
    switch (Op) {
    case '+':
      return builder->CreateFAdd(L,R,"addres");
    case '-':
      return builder->CreateFSub(L,R,"subres");
    case '*':
      return builder->CreateFMul(L,R,"mulres");
    case '/':
      return builder->CreateFDiv(L,R,"addres");
    case '<':
      return builder->CreateFCmpULT(L,R,"lttest");
    case '=':
      return builder->CreateFCmpUEQ(L,R,"eqtest");
    case 'a':
      //And
      return builder->CreateLogicalAnd(L,R,"andres");
    case 'o':
      //Or
      return builder->CreateLogicalOr(L,R,"orres");
    default:  
      std::cout << Op << std::endl;
      return LogErrorV("Operatore binario non supportato");
    }
  };
```