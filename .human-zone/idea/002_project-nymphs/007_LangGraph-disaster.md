# LangGraph 災難

主流 LLM 開發框架 LangChain 主要運作方式為 [ReAct 模式](./006_ReAct.md)，但是其線性思考以及不穩定性讓創造出來的 Agent 很難控制。

為了解決這個問題它們發明了 LangGraph，透過引入 Graph 來解決 LangChain 的缺陷，然而其本質是 FSM (有限狀態機)。

FSM 因為其缺陷早在二十年前在遊戲產業就被行為樹所取代，因此 LangGraph 本質上是一個重蹈覆轍的災難。
